# 🚀 Cluster HPC TECHNE — Documentação Técnica Completa

Este repositório documenta a arquitetura, configuração e infraestrutura do **Cluster HPC TECHNE**, utilizado para processamento de alto desempenho (HPC) com gerenciamento via **Slurm**.

---

## 📌 1. Visão Geral e Arquitetura

O cluster TECHNE é composto por um nó controlador e múltiplos nós de execução heterogêneos, com suporte a GPUs NVIDIA L4, armazenamento compartilhado e monitoramento centralizado.

### 🔧 Componentes Principais

| Componente | Detalhes Técnicos |
|-----------|-------------------|
| **Controlador / Master** | `slurm-master` — IP: `10.xx.yy.zz`<br>Serviços: Slurmctld, Slurmdbd, PostgreSQL/MariaDB, Munge |
| **Nó de Execução 1** | `gpunode01` — 16 Cores, 62.9 GB RAM<br>2x GPUs NVIDIA L4 |
| **Nó de Execução 2** | `gpunode02` — 12 Cores, 31.0 GB RAM<br>1x GPU NVIDIA L4 |
| **Sistema Operacional** | Linux Ubuntu/Debian — Kernel 6.8.x |
| **Armazenamento** | NFS em `/data/` + LVM no disco principal |

### 🖥️ Configuração de Hardware (via `lshw`)

- **CPU:** Intel® Xeon® Gold 6526Y (2 sockets lógicos)  
- **RAM Total:** 32 GiB (62.9 GiB disponíveis ao Slurm via `RealMemory`)  
- **GPUs:** 2× NVIDIA L4 (AD104GL) — 23 GB VRAM cada  
- **Controladoras:** Virtio SCSI e Virtio Network  

---

## 📡 2. Configuração do Agendador Slurm

O Slurm é configurado de modo centralizado e replicado em todos os nós,
utilizando Munge para autenticação.

### 📄 2.1. `slurm.conf` (Trecho Principal)

``` ini
# Configuração Central
ControlMachine=slurm-master
ControlAddr=10.xx.xx.xx
SlurmUser=slurm
SlurmctldPort=6817
SlurmdPort=6818
AuthType=auth/munge

# Contabilidade
JobCompType=jobcomp/none
JobAcctGatherType=jobacct_gather/cgroup
AccountingStorageHost=slurm-master

# Definição de Nós e Partições
NodeName=gpunode01 NodeAddr=10.9x.xx.xx Gres=gpu:2 State=IDLE
NodeName=gpunode02 NodeAddr=10.9x.xx.xx Gres=gpu:1 State=IDLE

PartitionName=gpu_part Nodes=gpunode01,gpunode02 Default=YES
```

### 📊 2.2. Contabilidade e Logs

-   **slurmdbd** executando no controlador.
-   Bancos:
    -   **MariaDB** → Slurm Accounting\
    -   **PostgreSQL** → Métricas do monitoramento / Grafana
-   Usuário **manager** com `AdminLevel=Manager` no `sacctmgr`.

------------------------------------------------------------------------

## 📈 3. Pipeline de Monitoramento (Customizado)

O cluster possui um pipeline próprio de coleta e visualização de
métricas via Python + PostgreSQL + Grafana.

### 🐍 3.1. Agente Python (`collect_metrics.py`)

-   Local: `/opt/cluster_monitor/collect_metrics.py`
-   Execução: a cada **1 minuto** (via `cron`)
-   Funções:
    -   coleta completa do estado do cluster (jobs, GPU, CPU, RAM)
    -   normalização dos dados
    -   envio ao PostgreSQL

#### ✔ Correção Importante

Para compatibilidade com CUDA:

``` bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib/x86_64-linux-gnu
```

Sem isso, PyTorch não encontra as bibliotecas CUDA.

### 🗄️ 3.2. Estrutura do Banco (PostgreSQL)

  -----------------------------------------------------------------------
  Tabela                   Armazena                       Uso
  ------------------------ ------------------------------ ---------------
  **gpu_log**              Utilização, memória usada,     Gráficos
                           temperatura                    temporais de
                                                          GPU

  **job_log**              Histórico de jobs (JobID,      Auditoria e
                           runtime, state)                estatísticas

  **queue_state**          Contagem de jobs por estado    Status da fila
                                                          no Grafana

  **utilization**          Uso de CPU (%) e RAM (%) por   Painéis de
                           nó                             ocupação
  -----------------------------------------------------------------------

### 📊 3.3. Dashboards no Grafana

-   **Jobs por Estado** → Gráfico de barras
-   **Uso da GPU** → Time series (GPU 0 / GPU 1)
-   **Uso de Disco** → Gauge (percentual)

------------------------------------------------------------------------

## 🧩 4. Tecnologias Utilizadas (Active Stack)

  Categoria             Tecnologias
  --------------------- -------------------------------------------------
  **Gerenciamento**     Slurm 23.11.4, Munge, systemd
  **Aceleração**        NVIDIA Drivers 570.x, CUDA 12.0/12.8, cuDNN 8.9
  **Desenvolvimento**   Python 3.12, venv
  **Bibliotecas**       psycopg2, psutil, subprocess, re, python-dotenv
  **Bancos de Dados**   PostgreSQL, MariaDB
  **Rede**              SSH/SCP, ufw

------------------------------------------------------------------------

## 📚 Licença

Este documento faz parte da infraestrutura interna do Cluster TECHNE e
pode ser reutilizado para estudos, configuração e expansão do ambiente.

------------------------------------------------------------------------

## ✨ Contato

**INFRA NCAD / UFPI**\
Gerenciamento e Desenvolvimento do Cluster HPC TECHNE

