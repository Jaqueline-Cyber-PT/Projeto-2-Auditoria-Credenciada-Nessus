# Gestão de Vulnerabilidades e Auditoria Credenciada-Fase2

### 1. Introdução: O Próximo Nível de Visibilidade

Após concluir o reconhecimento externo na Fase 1, onde validei que o **PF-DC01** possui uma superfície de ataque inicial protegida contra enumeração anónima, a análise evolui agora para uma **Auditoria Credenciada**. O objetivo é ultrapassar a "porta fechada" e avaliar o sistema por dentro, verificando falhas de *patching*, configurações de registo e vulnerabilidades de software que não são visíveis numa varredura externa.

### 2. Preparação do Alvo (Windows Server)

Para que a auditoria seja profunda, preparei o controlador de domínio para permitir o acesso do scanner. Criei uma conta de serviço dedicada, o utilizador **`NessusService`**, atribuindo-lhe privilégios de Administrador do Domínio. Além disso, ativei o serviço de **Remote Registry**, permitindo que o Nessus consulte as chaves de configuração do Windows.

<img width="1117" height="345" alt="image" src="https://github.com/user-attachments/assets/2f12d3fd-68ce-486f-9877-3dd23e3331c0" />

### 3. Implementação da Ferramenta de Auditoria (Kali Linux)

No lado do atacante (Kali), instalei o **Nessus Essentials**. Este processo envolveu a instalação do pacote `.deb`, o registo da licença e a sincronização da base de dados de *plugins* (assinaturas de vulnerabilidades atualizadas).

- **Comando de Instalação:** `sudo dpkg -i Nessus-10.11.2-ubuntu1604_amd64.deb`
- **Inicialização do Serviço:** `sudo systemctl start nessusd`

<img width="815" height="167" alt="image" src="https://github.com/user-attachments/assets/5e0c9059-183a-4f62-aa5c-d49e837db385" />

### 4. Configuração do Scan Credenciado

Com o motor do Nessus operacional através da interface `https://localhost:8834`, configurei um **Advanced Network Scan**. O diferencial técnico desta etapa é a inserção das credenciais do `NessusService`, permitindo que o scanner realize uma inspeção "com chaves na mão".

### 5. Análise de Resultados e Descobertas

O scan credenciado revelou um total de **61 vulnerabilidades agrupadas**, um salto significativo em comparação com a visibilidade externa inicial.

<img width="1386" height="442" alt="image" src="https://github.com/user-attachments/assets/97359800-0268-49da-b297-5b5c2db0810b" />

Este resultado confirma que, embora o perímetro (SMB) estivesse com um hardening básico, o interior do sistema carece de uma política rigorosa de atualização de software.

### 6. Verificação Manual e Exploração de Identidade

Bloqueio de LDAP Signing: Após o scan automático do Nessus, decidi "sujar as mãos" e validar manualmente o que um atacante veria se tivesse acesso a uma conta de serviço básica. O objetivo foi testar a resistência das políticas de grupo e a visibilidade de dados sensíveis.

Tentei mapear o Active Directory usando o **BloodHound**, mas o servidor barrou a conexão. Usei o **Comando:** `bloodhound-python -u NessusService -p 'Senha' -d pixelforge.internal -ns 192.168.1.150 -c All &> log_bloodhound.txt` mas sem sucesso, o resultado apresentou falha por exigência de **LDAP Signing**. Uma excelente notícia para a defesa, pois impede a extração simples de dados via LDAP.

<img width="910" height="356" alt="image" src="https://github.com/user-attachments/assets/0cc79099-d683-4cbd-8839-980a2348c818" />

Auditoria via RPC/SMB: Como um bom auditor não desiste, mudei o protocolo. Utilizei o Enum4Linux-ng para extrair informações via RPC, onde havia maior superfície de ataque. Usando o comando enum4linux-ng -A -u "NessusService" -p "Senha" 192.168.1.150 > auditoria_rpc_completa.txt 2>&1 obtive sucesso total, extraindo toda a Política de Senhas (tamanho mínimo, complexidade) e a Lista Completa de Usuários.

<img width="952" height="317" alt="image" src="https://github.com/user-attachments/assets/a08c799e-da0a-4387-8f55-ebc10a1cb579" />

### 7. Plano de Remediação Sugerido

Com base nos achados, as seguintes ações são recomendadas para elevar o nível de segurança do **PF-DC01**:

<img width="788" height="475" alt="image" src="https://github.com/user-attachments/assets/e8ed0e32-561d-4af0-964f-014791d1a2ff" />

### 8. Conclusão do Projeto e próximos passos

Esta auditoria demonstrou que ferramentas automáticas como o **Nessus** são fundamentais para visibilidade, mas a validação manual foi o que revelou a real fragilidade do domínio. Identificamos que defesas modernas (como o **LDAP Signing**) são insuficientes se protocolos legados (como o **RPC**) ainda permitem a extração de informações críticas.

A maior vulnerabilidade encontrada não foi um software desatualizado, mas sim a **política de senhas de 7 caracteres**, provando que a segurança de um ambiente depende tanto de *patches* quanto de um *hardening* rigoroso das configurações de GPO.

Com as vulnerabilidades de patch e as fragilidades de GPO (política de senhas) mapeadas, o próximo estágio deste laboratório focará na **Exploração e Movimentação Lateral**. Utilizaremos os dados extraídos via RPC para realizar ataques de *Brute Force* direcionados e testar a eficácia dos controles de detecção do Windows Server, simulando o comportamento real de um atacante após a fase de reconhecimento.

"Observação: Devido às limitações da licença Essentials, a exportação detalhada foi consolidada através do dashboard de vulnerabilidades e logs técnicos, cujas evidências estão anexadas na pasta /evidencias."


