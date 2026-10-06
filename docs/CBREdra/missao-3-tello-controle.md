# Levantamento de Requisitos de Engenharia
## Missão 3: Interação Humano-Robô (HRI)

* **Organização:** Equipe EDRA de Robótica Aérea
* **Subequipe:** Controle / Software
* **Plataforma-Alvo:** DJI Tello
* **Documento Base:** Edital Flying EDRA League 2026 (RoboCup Brasil - FRL)
* **Status:** Versão Preliminar para Revisão Interna

---

## 1. Padrão de Nomenclatura e Classificação

Os requisitos deste documento seguem o formato de indexação estruturado:

$$\mathbf{[REQ\text{-}CATEGORIA\text{-}ID]}$$

Onde:
* **CATEGORIA:**
  * `HW`: Requisitos de Hardware e Plataforma Mecânica/Elétrica
  * `OP`: Requisitos Operacionais, Regulatórios e de Arena
  * `SW`: Requisitos Funcionais de Software, Visão e Controle
  * `SC`: Regras de Pontuação, Sucesso e Penalidades (*Scoring*)
* **Nível de Criticidade:**
  * `[MANDATÓRIO]`: Requisito indispensável; a não conformidade resulta em desclassificação imediata ou nota zero.
  * `[ESTRATÉGICO]`: Requisito de alto impacto no resultado e na pontuação final.
  * `[OPERACIONAL]`: Requisito de procedimento ou restrição de tempo/espaço.

---

## 2. Requisitos de Hardware e Plataforma (HW)

| Identificador | Título / Requisito | Criticidade | Especificação Técnica e Restrições | Conformidade no DJI Tello |
| :--- | :--- | :---: | :--- | :--- |
| **`REQ-HW-01`** | **Distância entre Eixos** | `[MANDATÓRIO]` | A distância máxima entre os eixos das hélices deve ser de no máximo $330\text{ mm}$. Hélices com diâmetro máximo de $7''$. | **Conforme:** O Tello possui entre-eixos diagonal de $\sim 92\text{ mm}$ e hélices de $3''$. |
| **`REQ-HW-02`** | **Protetores de Hélices** | `[MANDATÓRIO]` | Obrigatório o uso de protetores de hélice (*prop guards*), devendo estar alinhados na altura do plano de rotação. | **Conforme:** Instalação obrigatória dos protetores plásticos originais/compatíveis. |
| **`REQ-HW-03`** | **Corte de Emergência** | `[MANDATÓRIO]` | O sistema deve possuir mecanismo de interrupção imediata dos motores (*kill switch*) acessível pelo operador ao comando da Banca. | **Conforme:** Implementado via comando `emergency` (SDK) ou rotina dedicada no nó de teleoperação. |
| **`REQ-HW-04`** | **Classificação de Hardware** | `[OPERACIONAL]` | Drones comerciais prontos (DJI/Tello) não se enquadram como *open-hardware*, anulando o multiplicador de $2\times$ desta categoria. | **Restrição Registrada:** A estratégia foca na maximização de pontos de software/tarefa. |
| **`REQ-HW-05`** | **Isolamento Físico (Sem Cabos)** | `[MANDATÓRIO]` | Proibido qualquer tipo de cabo, fio ou cordão umbilical para força, controle ou telemetria em voo. | **Conforme:** Voo sustentado por bateria interna e link sem fio Wi-Fi (UDP). |

---

## 3. Requisitos Operacionais e de Arena (OP)

| Identificador | Título / Requisito | Criticidade | Especificação Técnica e Restrições |
| :--- | :--- | :---: | :--- |
| **`REQ-OP-01`** | **Janela de Tentativas** | `[OPERACIONAL]` | A equipe dispõe de até $30\text{ minutos}$ totais por rodada para executar até $3\text{ tentativas}$. |
| **`REQ-OP-02`** | **Tempo Limite de Voo** | `[OPERACIONAL]` | Cada tentativa tem duração máxima permitida de **$15\text{ minutos}$** de relógio corrido. |
| **`REQ-OP-03`** | **Topologia de Frota** | `[ESTRATÉGICO]` | A equipe pode optar por **1 drone** (deve pousar em todas as 6 bases) ou **2 drones** sequenciais/simultâneos (cada robô deve pousar em 3 bases exclusivas). |
| **`REQ-OP-04`** | **Posicionamento do Piloto Humano** | `[MANDATÓRIO]` | O membro responsável pelos gestos deve permanecer estritamente no centro da arena (marcado com 'X'), em qualquer orientação inicial. |
| **`REQ-OP-05`** | **EPIs Mandatórios** | `[MANDATÓRIO]` | O operador dentro da arena deve trajar: calça comprida, camisa de manga longa, calçado fechado, capacete, óculos de segurança, luvas anti-corte e colete refletivo. |
| **`REQ-OP-06`** | **Gatilhos de Encerramento** | `[MANDATÓRIO]` | A tentativa é imediatamente abortada e encerrada se o robô pousar fora da base demarcada, pousar na base de decolagem ou se houver intervenção manual via rádio. |

---

## 4. Requisitos de Software, Controle e Visão (SW)

### 4.1. Sequência e Fluxo de Voo

| Identificador | Fase do Voo | Criticidade | Comportamento Requerido do Algoritmo |
| :--- | :--- | :---: | :--- |
| **`REQ-SW-01`** | **Decolagem Autônoma** | `[MANDATÓRIO]` | O robô deve realizar a decolagem inicial através de comando do computador (via terminal/script), sem uso de controle remoto manual. |
| **`REQ-SW-02`** | **Aproximação de Enquadramento** | `[ESTRATÉGICO]` | Após decolar, o drone deve navegar autonomamente até o operador no centro para enquadrá-lo no Campo de Visão (*FoV*) ótimo da câmera. |
| **`REQ-SW-03`** | **Validação de Presença Humana** | `[MANDATÓRIO]` | **Gatekeeper:** O algoritmo deve reconhecer o operador e exibir confirmação no terminal antes de ir às bases. Se não reconhecido, **zero pontos** são computados. |
| **`REQ-SW-04`** | **Pouso nas Bases** | `[MANDATÓRIO]` | O robô deve conduzir a aproximação e pousar com todo o trem de pouso apoiado na plataforma e desligar completamente os motores. |
| **`REQ-SW-05`** | **Retorno Autônomo** | `[ESTRATÉGICO]` | Ao terminar as visitas, o drone deve retornar e pousar na base inicial de decolagem sem intervenção humana para dobrar a pontuação. |

### 4.2. Diretrizes e Vedações Algorítmicas

| Identificador | Diretriz / Restrição | Criticidade | Descrição Detalhada da Restrição |
| :--- | :--- | :---: | :--- |
| **`REQ-SW-06`** | **Condução Exclusiva por Gestos** | `[MANDATÓRIO]` | Toda navegação direcional/angular deve ser derivada unicamente de ações corporais ou gestos visuais do operador. |
| **`REQ-SW-07`** | **Proibição de Rastreamento (Follow-Me)** | `[MANDATÓRIO]` | **Expressamente vedado:** Uso de algoritmos de rastreamento passivo de pessoas (*person following* / detecção de contorno contínuo para seguir o operador). |
| **`REQ-SW-08`** | **Invalidação de Marcadores de Base** | `[MANDATÓRIO]` | As bases estarão sem marcadores conhecidos (virados ou ocultos). Proibido usar coordenadas fixas pré-programadas ou mapeamento prévio de bases. |

### 4.3. Interface, Auditoria e Logs

| Identificador | Tópico de Log | Criticidade | Saída Requerida em Execução |
| :--- | :--- | :---: | :--- |
| **`REQ-SW-09`** | **Exibição do Rastreamento Corporal** | `[MANDATÓRIO]` | A tela/terminal de execução deve expor os dados de detecção visual (ex.: esqueleto de pose, coordenadas do operador no espaço ou mapa de calor). |
| **`REQ-SW-10`** | **Log de Comandos em Tempo Real** | `[MANDATÓRIO]` | O nome de cada gesto identificado deve ser impresso no terminal no momento da transição (ex.: `CMD: TRANSLATE_LEFT`, `CMD: HOVER`, `CMD: LAND`). |
| **`REQ-SW-11`** | **Gravação de Telemetria (Rosbags)** | `[OPERACIONAL]` | Recomendação obrigatória de gravação contínua dos tópicos de vídeo, comandos de velocidade (`cmd_vel`) e estados para auditoria da Banca. |

---

## 5. Matriz de Requisitos de Pontuação (SC)

| Identificador | Evento / Condição | Criticidade | Efeito no Score | Regra do Regulamento |
| :--- | :--- | :---: | :---: | :--- |
| **`REQ-SC-01`** | **Detecção do Operador** | `[MANDATÓRIO]` | Condição base | Pré-requisito: se não demonstrado no terminal, nenhuma pontuação é validada. |
| **`REQ-SC-02`** | **Visita a Base Inédita** | `[ESTRATÉGICO]` | **$+20\text{ pontos}$** | Concedido a cada nova base pousada com estabilização e motores desligados. |
| **`REQ-SC-03`** | **Pouso Repetido (Mesmo Drone)** | `[OPERACIONAL]` | **$-5\text{ pontos}$** | Penalidade por reincidência de pouso em uma base já visitada pela própria aeronave. |
| **`REQ-SC-04`** | **Invasão de Base (Caso Dual Drone)** | `[OPERACIONAL]` | **$-10\text{ pontos}$** | Penalidade se um drone pousar na base previamente visitada pelo outro drone. |
| **`REQ-SC-05`** | **Pouso Fora da Plataforma** | `[MANDATÓRIO]` | **Encerramento** | O pouso em qualquer área não autorizada finaliza a tentativa imediatamente. |
| **`REQ-SC-06`** | **Retorno Autônomo com Sucesso** | `[ESTRATÉGICO]` | **Multiplicador $2\times$** | Se pousar na base de decolagem sem rádio manual após $\ge 1$ base visitada, a pontuação positiva dobra. |

---

## 6. Checklist Rápido de Pré-Voo

- [ ] Protetores de hélice montados e firmes (`REQ-HW-02`)
- [ ] Bateria do Tello carregada ($>80\%$) e link Wi-Fi validado (`REQ-HW-05`)
- [ ] Operador equipado com todos os 7 itens de EPI no ponto central (`REQ-OP-04`, `REQ-OP-05`)
- [ ] Terminal configurado para exibição contínua do esqueleto e comando interpretado (`REQ-SW-09`, `REQ-SW-10`)
- [ ] Gravação de log/rosbag ativada em background (`REQ-SW-11`)
- [ ] Rotina de decolagem pronta para disparo por software (`REQ-SW-01`)