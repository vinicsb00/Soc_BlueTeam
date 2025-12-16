# Laboratório Prático — Wazuh(SIEM) + Fail2Ban + Simulação de Ataque SSH Brute Force

## Visão geral

Este relatório documenta, de forma prática e detalhada, a construção de um laboratório de segurança defensiva (Blue Team) utilizando Wazuh, Fail2Ban e a simulação controlada de um ataque SSH brute force com Hydra.

O foco não é apenas apresentar o resultado final, mas registrar todo o processo, desde a criação das máquinas virtuais até a observação e mitigação do ataque, incluindo decisões técnicas, configurações e validações realizadas.

O material foi pensado para ser compreensível por estudantes de segurança da informação, ao mesmo tempo em que mantém coerência técnica para profissionais da área.

---

## Objetivos do laboratório

- Criar um ambiente realista de monitoramento de segurança
- Entender, na prática, o funcionamento de um SIEM distribuído
- Simular um ataque real de força bruta via SSH
- Observar logs e eventos de segurança
- Implementar mitigação local seguindo boas práticas
- Conectar teoria com prática operacional

---

## Planejamento do ambiente

O laboratório foi estruturado com três papéis bem definidos:

- Um servidor exclusivo para monitoramento (SIEM)
- Um host separado que seria monitorado e atacado
- Uma máquina dedicada à simulação do atacante

Essa separação reflete ambientes reais, onde serviços, monitoramento e ataque nunca estão concentrados no mesmo host.

---

## ISOs utilizadas

As imagens ISO foram obtidas diretamente dos sites oficiais:

- Ubuntu Server 22.04 LTS — servidor Wazuh
- Ubuntu Desktop 22.04 LTS — host monitorado
- Kali Linux (Installer) — máquina atacante

A escolha por versões LTS foi feita visando estabilidade e compatibilidade com as ferramentas utilizadas.

---

## Virtualização e criação das VMs

O ambiente foi virtualizado utilizando VirtualBox.

### Configuração das máquinas virtuais

Ubuntu Server — Wazuh Server
- RAM: 4 GB
- CPU: 2 vCPUs
- Disco: 50 GB
- Interface gráfica: não

Ubuntu Desktop — Host monitorado
- RAM: 4 GB
- CPU: 2 vCPUs
- Disco: 30 GB

Kali Linux — Atacante
- RAM: 4 GB
- CPU: 2 vCPUs
- Disco: 30 GB

Essas configurações foram suficientes para executar o laboratório de forma estável.

---

## Configuração de rede

Todas as máquinas virtuais foram configuradas em modo NAT.

Essa escolha permitiu:
- Comunicação entre as VMs
- Acesso à internet para instalação de pacotes
- Simplicidade na configuração inicial do ambiente

---

## Ajustes iniciais no Ubuntu Desktop

Após a instalação do Ubuntu Desktop, foi necessário ajustar o layout do teclado para o padrão ABNT2:
````bash
setxkbmap -model pc105 -layout br -variant abnt2
````
---

## Instalação do Wazuh Server

No Ubuntu Server, a instalação do Wazuh foi feita utilizando o script oficial:
````bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
````
````bash
sudo bash wazuh-install.sh -a --ignore-check
````
O parâmetro --ignore-check foi necessário devido a algumas verificações automáticas que não eram compatíveis com o ambiente virtualizado, permitindo a instalação completa sem erros.

---

## Validação dos serviços do Wazuh

Após a instalação, todos os serviços foram verificados manualmente:

````bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
````
O estado RUNNING indica:

- Serviço iniciado corretamente
- Processo ativo
- Ausência de falhas críticas

Caso fosse necessário, os serviços podiam ser iniciados manualmente:

````bash
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-dashboard
````
O acesso ao Wazuh Dashboard foi feito via navegador:

https://IP_DO_SERVER

---

## Instalação do Wazuh Agent

No Ubuntu Desktop (host monitorado), foi instalado o Wazuh Agent, permitindo a coleta de logs e eventos:

````bash
sudo apt install wazuh-agent -y
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
````
---

## Instalação e ativação do SSH

No host monitorado, o SSH foi instalado para possibilitar a simulação do ataque:

````bash
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
````
---

## Simulação do ataque SSH (Hydra)

No Kali Linux, foi executada uma simulação controlada de brute force:

````bash
hydra -l usuario -P senhas.txt ssh://IP_DO_AGENTE -t 1
````
O uso de apenas uma thread foi intencional para:

- Manter o ataque controlado
- Facilitar observação dos logs
- Evitar sobrecarga no ambiente

O objetivo foi gerar eventos reais, não comprometer credenciais.

---

## Observação dos eventos no Wazuh

Durante o ataque, o Wazuh Dashboard apresentou:

- Falhas de autenticação SSH
- IP de origem
- Usuário alvo
- Tentativas repetidas (~500)

Isso confirmou que:

- O agente estava coletando logs corretamente
- O manager estava processando os eventos
- O SIEM estava funcionando como esperado

---

## Implementação do Fail2Ban

No host monitorado, o Fail2Ban foi instalado:

````bash
sudo apt install fail2ban -y
````
Arquivo de configuração:

nano /etc/fail2ban/jail.local

Configuração aplicada:
````bash
[sshd]
enabled = true
backend = systemd
port = ssh

maxretry = 3
findtime = 60

bantime = 600
bantime.increment = true
bantime.factor = 2
bantime.maxtime = 86400
````
Essa configuração aplica bloqueios progressivos e reduz impacto de falsos positivos.

---

## Resultados observados

Após ativar o Fail2Ban:

- Conexões do Hydra foram bloqueadas
- Número de tentativas caiu drasticamente
- IPs foram temporariamente banidos
- Logs mostraram mensagens como maximum authentication attempts exceeded

---

## Conclusão

Este laboratório demonstrou que:

- Ataques geram eventos reais
- O SIEM observa e correlaciona
- A mitigação ocorre no host final
- Defesa em camadas é essencial

O foco não foi apenas executar ferramentas, mas compreender todo o processo defensivo, reforçando raciocínio de Blue Team.

---
