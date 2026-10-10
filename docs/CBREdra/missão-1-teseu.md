**Equipe TESEU**

# **Missão 1 — LOCALIZAÇÃO E MAPEAMENTO**

1 INTRODUÇÃO

Este documento consolida os requisitos de sistema, a arquitetura de referência de controle e autonomia, bem como o plano de verificação e validação para a Fase 1 — Localização e Mapeamento. A missão consiste em decolar de forma autônoma, navegar com precisão em ambiente interno sem sinal de GPS, identificar bases de pouso móveis e fixas, executar pousos com desarme/rearme dos motores em bases distintas e retornar à base de origem de modo 100% autônomo para maximizar a pontuação da tentativa.

1.1 Escopo

O sistema compreende a aeronave multirrotor, subsistema de propulsão e alimentação, controlador de voo (flight controller), computador de bordo (companion computer), sensores de odometria local (IMU, fluxo óptico e telêmetros LiDAR), câmera orientada para o solo, middleware ROS 2, máquina de estados/árvore de comportamento e estação de controle em solo (GCS).

1.2 Premissas e restrições da missão

| ID | Condição |
| :--- | :--- |
| EXT-01 | Ambiente interno fechado (indoor) com negação total de sinal GPS. |
| EXT-02 | Bases de pouso na arena com elevação de até 1,5 m de altura em relação ao solo. |
| EXT-03 | Visita obrigatória a 2 bases distintas com validação por desarme completo (*disarm*). |
| EXT-04 | Proibição de repetição ou mistura de bases (penalidade por pouso repetido). |
| EXT-05 | Todo o trem de pouso deve tocar a superfície da base de forma alinhada e contida. |
| EXT-06 | Retorno autônomo à base de decolagem confere multiplicador de pontuação (2x). |

1.3 Objetivo de engenharia

Garantir navegação estável e imune a derivas acumuladas em ambiente privado de GNSS, integrando visão reativa e controle em malha fechada para realizar o ciclo de visitas e retorno com repetibilidade e sem intervenção humana.

---

2 SYSTEM REQUIREMENTS REVIEW (SRR)

O SRR define o comportamento esperado do sistema de controle e autonomia para o cumprimento integral da Fase 1.

| ID | Requisito | Pri. | V |
| :--- | :--- | :--- | :--- |
| SYS-01 | Decolagem vertical autônoma e estabilizada. | P0 | D |
| SYS-02 | Manutenção de altitude de cruzeiro na faixa de 1,5 m a 2,0 m. | P0 | T |
| SYS-03 | Estabilização rigorosa de deslocamento em ambiente sem GPS. | P0 | T |
| SYS-04 | Navegação direcionada a coordenadas pré-estabelecidas (Base 1). | P0 | D |
| SYS-05 | Alinhamento e centralização reativa com a base via visão. | P0 | T |
| SYS-06 | Pouso estritamente vertical e desarme (*disarm*) completo. | P0 | T |
| SYS-07 | Rearme autônomo e decolagem sequencial pós-pouso. | P0 | D |
| SYS-08 | Filtragem e distinção de alvos para evitar repetição de bases. | P0 | T |
| SYS-09 | Retorno autônomo e pouso na base de decolagem (*takeoff*). | P0 | D |

*Legenda de verificação: I = inspeção; A = análise; T = teste; D = demonstração.*

### 2.1 Detalhamento dos Requisitos da Missão

* **SYS-01 — Decolagem vertical:** O drone deve ser capaz de armar os motores e ganhar altitude de forma autônoma e estabilizada, saindo da inércia no solo de maneira segura sem interferência humana.
* **SYS-02 — Atingir altura esperada (1,5 m ~ 2,0 m):** A altitude de cruzeiro deve ser mantida nessa faixa para evitar colisões frontais com as bases móveis (que podem atingir até 1,5 m de altura), garantindo simultaneamente um campo de visão adequado para a câmera.
* **SYS-03 — Controle preciso de deslocamento:** A navegação pela arena exige estabilização rigorosa. Por ser um ambiente sem GPS, a malha de controle depende da fusão de sensores inerciais (IMU), fluxo óptico e telêmetros (LiDAR) para evitar derivas que causem colisões nas paredes da arena.
* **SYS-04 — Capacidade de chegar ao local pré-estabelecido:** O sistema de navegação global deve ser capaz de receber as coordenadas de uma base conhecida (Base 1) e traçar uma rota direta e otimizada até ela.
* **SYS-05 — Alinhamento e centralização com a base:** Ao detectar uma base de pouso, o drone deve alternar para um controle reativo baseado em visão, centralizando o alvo no campo de visão e reduzindo os erros inerciais (X e Y) antes de iniciar a descida.
* **SYS-06 — Pouso e desarme:** A descida deve ser perfeitamente vertical para que todo o trem de pouso toque a base de forma alinhada. Imediatamente após o toque, o controlador de voo deve executar o *disarm* das hélices para que a visita à base seja validada pelas regras da fase.
* **SYS-07 — Rearme e início da próxima parte da missão:** Após a confirmação do pouso, o sistema de controle deve rearmar os motores autonomamente e realizar uma nova decolagem, retomando a rotina de navegação para procurar o próximo alvo.
* **SYS-08 — Busca sem confusão, mistura de bases:** A Behavior Tree deve implementar uma lógica para ignorar a base em que o drone acabou de pousar. Isso evita penalidades por pousos repetidos.
* **SYS-09 — Retorno à base de takeoff:** Após validar o pouso nas duas bases exigidas, o drone deve planejar e executar o trajeto de volta para a base inicial, garantindo o multiplicador de pontos (2x) por concluir a missão de forma totalmente autônoma.

---

3 DESIGN REQUIREMENTS DOCUMENT (DRD)

Baseline técnica para suporte aos requisitos de controle, fusão sensorial e decisão.

| Subsistema | Baseline | Função |
| :--- | :--- | :--- |
| Controle de Baixo Nível | Pixhawk com firmware PX4 | Estabilização angular, malha de atitude e fusão EKF2. |
| Computador de Bordo | Companion computer | Execução dos nós ROS 2, processamento de imagem e Behavior Tree. |
| Sensoriamento Indoor | Câmera de fluxo óptico + IMU | Estimativa de velocidade planar (Vx, Vy) em arena sem GNSS. |
| Telemetria de Altura | Sensor LiDAR / ToF unidirecional | Medição contínua de distância real ao solo e topo das bases. |
| Visão Computacional | Câmera monocular voltada para baixo | Rastreamento e estimativa de pose relativa da base de pouso. |
| Middleware e Comunicação | ROS 2 + micro-XRCE-DDS | Envio de *setpoints* de posição/velocidade em modo Offboard. |
| Tomada de Decisão | Behavior Tree / Máquina de Estados | Coordenação do ciclo: Decolagem → Rota → Pouso → Rearme → Retorno. |

---

4 REQUISITOS DE INTERFACE E SEGURANÇA

| ID | Requisito | Critério Resumido |
| :--- | :--- | :--- |
| INT-01 | Interface de comandos Offboard padronizada. | Envio periódico de setpoints em ROS 2 sem bypass das proteções nativas do PX4. |
| INT-02 | Validação de transição de estado da Behavior Tree. | Mudança de nó de controle condicionada a confirmação explícita de *disarm*/*arm*. |
| SAF-01 | Teto e piso de segurança de altitude. | Bloqueio de voo fora da faixa de 1,5 m a 2,0 m durante translado. |
| SAF-02 | Failsafe de perda de odometria visual/inercial. | Deriva acima da tolerância aborta a rota e força pouso vertical suave imediato. |
| SAF-03 | Interrupção supervisionada via GCS. | Chave física/lógica de segurança na GCS para corte de modo autônomo a qualquer momento. |

---

5 VERIFICAÇÃO E VALIDAÇÃO (V&V)

### 5.1 Campanhas de Ensaio em Simulação (Gazebo)

| ID | Ensaio | Critério de Aprovação |
| :--- | :--- | :--- |
| GZ-01 | Decolagem estabilizada | Subida vertical limpa até a faixa de 1,5 m ~ 2,0 m sem oscilações. |
| GZ-02 | Estabilidade sem GNSS | Navegação planar sustentada apenas por odometria simulada (fluxo/IMU/LiDAR). |
| GZ-03 | Trânsito até coordenadas da Base 1 | Chegada ao ponto pré-definido com desvio inferior à tolerância de busca. |
| GZ-04 | Alinhamento visual reativo | Centralização da base na imagem com minimização dos erros em X e Y. |
| GZ-05 | Pouso vertical e *disarm* | Toque com trem de pouso contido seguido de desarme imediato dos motores. |
| GZ-06 | Rearme e decolagem sequencial | Motores rearmam autonomamente e drone retoma altitude de cruzeiro. |
| GZ-07 | Rejeição de base visitada | Algoritmo ignora a base de onde acabou de sair e engaja busca da próxima. |
| GZ-08 | Retorno autônomo (*Return-to-Home*) | Trajetória reversa até a base de decolagem inicial com pouso final bem-sucedido. |

### 5.2 Matriz de Rastreabilidade V&V

| Requisito | Teste Associado | Ambiente | Evidência de Aceitação |
| :--- | :--- | :--- | :--- |
| SYS-01 | GZ-01 | Gazebo / Voo real | Logs de atitude e tempo de subida até estabilização. |
| SYS-02 | GZ-01 / GZ-02 | Gazebo / Voo real | Gráfico de altitude mantido na faixa de 1,5 m a 2,0 m. |
| SYS-03 | GZ-02 | Bancada / Voo real | Variância de posição planar sem auxílio de GPS. |
| SYS-04 | GZ-03 | Gazebo / Voo real | Registro de trajetória direta rumo à Base 1. |
| SYS-05 | GZ-04 | Gazebo / Voo real | Convergência dos erros nos eixos X e Y durante o tracking. |
| SYS-06 | GZ-05 | Gazebo / Voo real | Confirmação de desarme nos logs e vídeo de toque contido. |
| SYS-07 | GZ-06 | Gazebo / Voo real | Telemetria confirmando rearme e nova decolagem sem comando manual. |
| SYS-08 | GZ-07 | Gazebo / Voo real | Log da Behavior Tree evidenciando exclusão do alvo anterior. |
| SYS-09 | GZ-08 | Gazebo / Voo real | Execução completa de retorno e pouso na base inicial. |

---

6 MATRIZ DE RISCOS DA MISSÃO

| Risco Identificado | Severidade | Ação Mitigadora Principal |
| :--- | :--- | :--- |
| Deriva de odometria em ambiente sem GPS | Crítica | Fusão rigorosa de IMU, fluxo óptico e telêmetro LiDAR no estimador EKF2. |
| Colisão frontal com bases elevadas de até 1,5 m | Crítica | Travamento estrito da altitude de cruzeiro na faixa de 1,5 m a 2,0 m. |
| Pouso repetido na mesma base (penalização) | Alta | Lógica de máscara de exclusão e marcação de visita na Behavior Tree. |
| Falha de centralização durante a descida final | Alta | Malha de controle visual servoing reativa acoplada à leitura contínua de altura. |
| Desestabilização no rearme e nova decolagem | Alta | Verificação de atitude neutra no solo antes do comando autônomo de *arm*. |
