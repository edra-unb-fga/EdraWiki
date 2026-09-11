# Mapa da Documentação

Índice de **toda a documentação técnica** espalhada pelos repositórios da EDRA,
organizada por tema. A ideia é: você acha aqui **onde** está cada coisa, sem
precisar caçar repo por repo.

!!! info "Como este mapa funciona"
    Este é um **índice de ponteiros** — ele linka para a documentação nos repos de
    origem (nada é duplicado). Conforme o conteúdo for revisado, ele pode ser
    migrado para dentro da própria EdraWiki. Repos marcados com :material-lock:
    são privados (só membros da organização acessam).

    Última varredura da organização: **53 repositórios**.

---

## 🛰️ Simulação (PX4 · Gazebo · ROS 2)

| Repositório | O que tem |
|-------------|-----------|
| **[Simulação CBR2025](simulacao/cbr2025-arena-missao1.md)** (aqui na wiki) | Passo a passo completo: arena + missão 1 de detecção/pouso, com todos os problemas e soluções |
| [Arenas_Gazebo](https://github.com/edra-unb-fga/Arenas_Gazebo) :material-lock: | Arenas do Gazebo (branch `CBR2025` = arena da CBR 2025) |
| [gazebo-px4](https://github.com/edra-unb-fga/gazebo-px4) :material-lock: | Docker PX4+Gazebo · [`DOCKER.md`](https://github.com/edra-unb-fga/gazebo-px4/blob/main/docs/DOCKER.md) · [`LISTA_DE_ERROS.md`](https://github.com/edra-unb-fga/gazebo-px4/blob/main/docs/LISTA_DE_ERROS.md) |
| [gazebo_world](https://github.com/edra-unb-fga/gazebo_world) :material-lock: | Ambiente do Gazebo para a CBR |
| [arena_cbr_2024](https://github.com/edra-unb-fga/arena_cbr_2024) :material-lock: | Arquivos da arena da CBR 2024 |
| [simulation-setup](https://github.com/edra-unb-fga/simulation-setup) | Instalação das ferramentas de simulação (PX4 + ROS 2 Humble) |

## ⚙️ Setup · Instalação · Docker

| Repositório | O que tem |
|-------------|-----------|
| [Configuracoes_Basicas_para_Controle](https://github.com/edra-unb-fga/Configuracoes_Basicas_para_Controle) | Tutorial: [`INSTALAÇÃO.md`](https://github.com/edra-unb-fga/Configuracoes_Basicas_para_Controle/blob/main/INSTALA%C3%87%C3%83O.md) e [`MISSÃO_TESTE.md`](https://github.com/edra-unb-fga/Configuracoes_Basicas_para_Controle/blob/main/MISS%C3%83O_TESTE.md) |
| [docker-configuration](https://github.com/edra-unb-fga/docker-configuration) | Configuração base de Docker |
| [ros_humble_docker](https://github.com/edra-unb-fga/ros_humble_docker) | Container ROS 2 Humble |
| [Workspace_Template](https://github.com/edra-unb-fga/Workspace_Template) :material-lock: | Template de workspace ROS 2 da equipe |
| [capacitacao-docker](https://github.com/edra-unb-fga/capacitacao-docker) | 🐳 Capacitação de Docker para a equipe e trainees |

## 🔌 Ponte PX4 ↔ ROS 2 (uXRCE-DDS)

| Repositório | O que tem |
|-------------|-----------|
| [uXRCE-configuration](https://github.com/edra-unb-fga/uXRCE-configuration) | Configuração do Agent uXRCE-DDS (ponte PX4–ROS 2) |
| [uxrce-docker](https://github.com/edra-unb-fga/uxrce-docker) | Container do uXRCE-DDS |
| [rasp-config-sae](https://github.com/edra-unb-fga/rasp-config-sae) :material-lock: | Container Docker de conexão Rasp + PX4 (uXRCE + ROS 2) |
| [ROS2-T265-PX4](https://github.com/edra-unb-fga/ROS2-T265-PX4) · [MAVROS](https://github.com/edra-unb-fga/ROS2-T265-PX4-MAVROS) | Conversão de mensagens T265 ROS 2 → PX4 |

## 🎮 Controle · PX4 · Behavior Trees

| Repositório | O que tem |
|-------------|-----------|
| [Producao-do-relatorio-da-SAE](https://github.com/edra-unb-fga/Producao-do-relatorio-da-SAE) :material-lock: | **Baú de conhecimento** de Controle/Embarcados. Destaques: Arquitetura PX4, [Offboard posição×velocidade](https://github.com/edra-unb-fga/Producao-do-relatorio-da-SAE), Hierarquias e Failsafes, Sistemas de Coordenadas, pytrees |
| [LLM-docs](https://github.com/edra-unb-fga/LLM-docs) :material-lock: | Conceitos + biblioteca de prompts: offboard control, state machines, behavior trees, sensor fusion, mission planning |
| [Rastreio-das-Bases-com-ROS2](https://github.com/edra-unb-fga/Rastreio-das-Bases-com-ROS2) :material-lock: | Rastreio das bases com ROS 2 |

## 👁️ Visão Computacional (datasets · treino · detecção)

| Repositório | O que tem |
|-------------|-----------|
| [dataset_generator_m1](https://github.com/edra-unb-fga/dataset_generator_m1) | Gerador de dataset **sintético 2D** da missão 1 da SAE |
| [codigo-treinamento-modelo-parametrizado](https://github.com/edra-unb-fga/codigo-treinamento-modelo-parametrizado) :material-lock: | Infra de treino de modelos de visão (Local/Colab/Kaggle), 100% parametrizável |
| [base_detector](https://github.com/edra-unb-fga/base_detector) :material-lock: | Modelo treinado para detecção de base da CBR |
| [pipeline-criacao-dataset](https://github.com/edra-unb-fga/pipeline-criacao-dataset) :material-lock: | Pipeline de criação de dataset |
| [conversao_e_inferencia_NCNN](https://github.com/edra-unb-fga/conversao_e_inferencia_NCNN) :material-lock: | Conversão e inferência com NCNN |
| [inferencia_modelos_SAE](https://github.com/edra-unb-fga/inferencia_modelos_SAE) :material-lock: | Inferência dos modelos da SAE |
| [Visao-fundamentos-para-a-SAE](https://github.com/edra-unb-fga/Visao-fundamentos-para-a-SAE) :material-lock: | Fundamentos de visão computacional (validação de conceitos) |
| [GUI_de_testar_modelos_com_docker](https://github.com/edra-unb-fga/GUI_de_testar_modelos_com_docker) | GUI para testar modelos via Docker |
| [imageaug-example](https://github.com/edra-unb-fga/imageaug-example) :material-lock: | Exemplos de data augmentation |
| [Color-segmentation-statistics-demonstrator](https://github.com/edra-unb-fga/Color-segmentation-statistics-demonstrator) :material-lock: | Demonstrador de segmentação por cor |
| [capacitacao-OpenCv-Numpy](https://github.com/edra-unb-fga/capacitacao-OpenCv-Numpy) | Capacitação de OpenCV + NumPy |
| [CodeScanner](https://github.com/edra-unb-fga/CodeScanner) | Leitura de códigos de barra/QR (branch `Python_Version`) |

## 🍓 Hardware · Raspberry Pi · Câmera

| Repositório | O que tem |
|-------------|-----------|
| [rasp-config-cbr](https://github.com/edra-unb-fga/rasp-config-cbr) :material-lock: | Configurações da Raspberry Pi 5 para a CBR |
| [rasp-config-sae](https://github.com/edra-unb-fga/rasp-config-sae) :material-lock: | Configurações da Rasp para a SAE |
| [rasp-config](https://github.com/edra-unb-fga/rasp-config) | Configuração base da Raspberry |
| [rasp-camera-calibration](https://github.com/edra-unb-fga/rasp-camera-calibration) | Calibração de câmeras da RPI 5 com OpenCV |
| [rasp-dataset](https://github.com/edra-unb-fga/rasp-dataset) :material-lock: | Coleta de dataset pela câmera da Rasp |
| [realsense-docker](https://github.com/edra-unb-fga/realsense-docker) | Container para a câmera RealSense |
| [bancada_de_testes](https://github.com/edra-unb-fga/bancada_de_testes) :material-lock: | Bancada de testes |
| [benchAeroprop](https://github.com/edra-unb-fga/benchAeroprop) :material-lock: | Bancada de propulsão |

## 🏆 Missões — CBR (Campeonato Brasileiro de Robótica)

| Repositório | O que tem |
|-------------|-----------|
| [CBR_ws](https://github.com/edra-unb-fga/CBR_ws) :material-lock: | Workspace da CBR 2025 (missões) |
| [Fase-3-CBR-Nathan-](https://github.com/edra-unb-fga/Fase-3-CBR-Nathan-) :material-lock: | Implementação avulsa da Fase 3 |
| [Fase_4_CBR_2025_Leonardo](https://github.com/edra-unb-fga/Fase_4_CBR_2025_Leonardo) :material-lock: | Fase 4: navegação autônoma em ambiente escuro/obstruído + pouso |
| [cbr2024_ws](https://github.com/edra-unb-fga/cbr2024_ws) · [cbr_ws_old](https://github.com/edra-unb-fga/cbr_ws_old) :material-lock: | Workspaces de CBR anteriores |
| [missoes-no-tello-2024](https://github.com/edra-unb-fga/missoes-no-tello-2024) :material-lock: · [tello_cbr](https://github.com/edra-unb-fga/tello_cbr) :material-lock: · [Tello](https://github.com/edra-unb-fga/Tello) | Missões com o drone Tello |

## 🏁 Missões — SAE (EletroQuad)

| Repositório | O que tem |
|-------------|-----------|
| [SAE26_ws](https://github.com/edra-unb-fga/SAE26_ws) :material-lock: · [SAE_ws](https://github.com/edra-unb-fga/SAE_ws) :material-lock: | Workspaces da competição SAE EletroQuad |
| [Producao-do-relatorio-da-SAE](https://github.com/edra-unb-fga/Producao-do-relatorio-da-SAE) :material-lock: | Conhecimento + escrita do relatório da SAE (ver seção Controle acima) |

## 📚 Institucional · Capacitação · Trainees

| Repositório | O que tem |
|-------------|-----------|
| [EdraDocs](https://edra-unb-fga.github.io/EdraDocs/) | **Site institucional** da equipe (história, membros, projetos, patrocinadores) |
| [Edital-do-trainee-2025-1](https://github.com/edra-unb-fga/Edital-do-trainee-2025-1) · [2024-2](https://github.com/edra-unb-fga/Edital-do-trainee-2024-2) | Editais e escopo do processo de trainees |
| [capacitacao-git](https://github.com/edra-unb-fga/capacitacao-git) | Capacitação de Git para trainees |
| [capacitacao-docker](https://github.com/edra-unb-fga/capacitacao-docker) | Capacitação de Docker |
| [capacitacao-OpenCv-Numpy](https://github.com/edra-unb-fga/capacitacao-OpenCv-Numpy) | Capacitação de OpenCV + NumPy |
| [LLM-docs](https://github.com/edra-unb-fga/LLM-docs) :material-lock: | Biblioteca de prompts para desenvolvimento com IA |

---

## 📋 Situação da migração

Conforme o conteúdo for revisado e trazido para a EdraWiki, marcamos aqui:

- [x] Simulação CBR2025 (Arena + Missão 1) — **migrado** ✅
- [x] Instalação nativa (Linux) — ROS 2, PX4, agente XRCE, workspace, QGC, arena — **migrado** ✅
- [x] Docker no Windows (WSL2) — **migrado** ✅
- [x] Inicialização desacoplada (fix timeout do gz_bridge) — **migrado** ✅
- [ ] Lista de erros comuns (a partir de `gazebo-px4/docs/LISTA_DE_ERROS.md`)
- [ ] Controle Offboard — posição × velocidade (a partir de `Producao-do-relatorio-da-SAE`)
- [ ] uXRCE-DDS — configuração da ponte
- [ ] Visão: criação de dataset sintético + treino

> Achou documentação que não está neste mapa? Adicione aqui via Pull Request —
> veja [Como contribuir](contribuir.md).
