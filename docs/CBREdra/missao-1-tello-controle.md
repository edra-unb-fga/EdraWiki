# Missão 1 — Localização e Mapeamento - Tello

## Organização dos requisitos

Os requisitos foram organizados de acordo com a função de cada subsistema dentro da execução da missão:

1. **Controle e Navegação (CTRL):** define as ações principais do UAV durante a missão, incluindo decolagem, deslocamento entre bases, pousos e retorno à base inicial.

2. **Malha de Navegação (NAV):** define requisitos relacionados à trajetória, precisão do deslocamento, aproximação e posicionamento sobre as bases.

3. **Controle de Estados da Missão (FSM):** define a sequência lógica da missão e as transições entre busca, aproximação, pouso, decolagem e retorno.

4. **Sincronização de Tempo e Comandos (TIME):** define a ordem e a temporização dos comandos necessários para a execução da missão.

---

## 1. Requisitos de Controle e Navegação (CTRL)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M1-CTRL-01** | O UAV deverá realizar a decolagem autonomamente a partir da base de decolagem. | Decolagem | Verificar a decolagem sem intervenção humana. |
| **REQ-M1-CTRL-02** | O UAV deverá percorrer a arena de forma autônoma durante a execução da missão. | Exploração da arena | Verificar a navegação sem pilotagem manual. |
| **REQ-M1-CTRL-03** | O sistema deverá permitir o deslocamento até a base cuja posição é previamente conhecida. | Navegação até base conhecida | Verificar se o UAV alcança a região correspondente à base informada. |
| **REQ-M1-CTRL-04** | Após receber do sistema de visão a localização de uma base desconhecida, o UAV deverá ser capaz de se deslocar até ela. | Busca → Navegação até base desconhecida | Simular ou utilizar a localização fornecida pelo módulo de visão e verificar o deslocamento correspondente. |
| **REQ-M1-CTRL-05** | O sistema deverá permitir o posicionamento do UAV sobre a base detectada para realização do pouso. | Aproximação da base | Verificar o alinhamento do UAV com a região de pouso. |
| **REQ-M1-CTRL-06** | O UAV deverá realizar pouso autônomo sobre cada base visitada. | Pouso em base | Verificar se o UAV completa o pouso sobre a base sem intervenção humana. |
| **REQ-M1-CTRL-07** | Após o pouso em uma base, o UAV deverá realizar nova decolagem para prosseguir com a missão. | Pouso → Decolagem | Verificar a retomada autônoma do voo após o pouso. |
| **REQ-M1-CTRL-08** | O sistema deverá evitar pousos repetidos em bases já visitadas. | Visita à base → Busca/Navegação para a próxima base | Registrar as bases já visitadas e verificar se o UAV não realiza novo pouso nessas bases durante a continuação da missão. |
|**REQ-M1-CTRL-09** | Após visitar as duas bases, o UAV deverá iniciar o retorno à base de decolagem. | Segunda base → Retorno | Verificar se, após a conclusão da visita à segunda base, o UAV inicia automaticamente o retorno à base de decolagem. |
| **REQ-M1-CTRL-10** | O UAV deverá ser capaz de retornar autonomamente à base de decolagem. | Retorno | Verificar o deslocamento autônomo até a base inicial. |
| **REQ-M1-CTRL-11** | O UAV deverá realizar pouso autônomo na base de decolagem ao final da missão. | Retorno → Pouso final | Verificar a conclusão da missão sobre a base inicial. |
| **REQ-M1-CTRL-12** | O sistema deverá permitir o retorno à base de decolagem mediante comando humano, conforme permitido pelo regulamento. | Retorno / Abortagem | Enviar o comando de retorno e verificar a resposta do UAV. |

---

## 2. Requisitos de Malha de Navegação (NAV)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M1-NAV-01** | O UAV deverá seguir a trajetória planejada entre a base de decolagem e as bases de pouso. | Navegação entre bases | Comparar a trajetória planejada com a executada. |
| **REQ-M1-NAV-02** | O sistema deverá permitir correções de trajetória durante o deslocamento até as bases. | Navegação | Introduzir ou observar desvios e verificar a correção do percurso. |
| **REQ-M1-NAV-03** | O UAV deverá manter erro de posicionamento compatível com a realização do pouso nas bases. | Aproximação / Pouso | Medir o erro de posição durante a aproximação. |
| **REQ-M1-NAV-04** | O sistema deverá permitir aproximação às bases localizadas em diferentes alturas dentro dos limites definidos pela missão. | Aproximação das bases | Realizar testes com bases posicionadas em diferentes alturas. |
| **REQ-M1-NAV-05** | O UAV deverá ajustar sua posição horizontal e vertical durante a aproximação da base. | Aproximação / Pouso | Verificar a execução dos ajustes antes do pouso. |
| **REQ-M1-NAV-06** | O sistema deverá conduzir o UAV de modo a minimizar deslocamentos desnecessários entre as bases. | Planejamento da rota | Comparar a rota executada com a sequência de pontos definida pelo sistema. |
| **REQ-M1-NAV-07** | O UAV deverá manter posição suficientemente estável antes de iniciar o pouso. | Aproximação final | Medir a variação da posição imediatamente antes do comando de pouso. |

---

## 3. Requisitos de Controle de Estados da Missão (FSM)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M1-FSM-01** | O sistema deverá implementar uma máquina de estados para controlar a sequência da Missão 1. | Toda a missão | Verificar a sequência de estados durante a execução. |
| **REQ-M1-FSM-02** | O sistema deverá possuir um estado inicial de preparação antes da decolagem. | Inicialização | Verificar o estado inicial da missão. |
| **REQ-M1-FSM-03** | Após a decolagem, o sistema deverá realizar a transição para o estado de ida à base conhecida. | Decolagem → Ida à Base Conhecida | Verificar a transição para o estado de navegação até a Base Conhecida após a decolagem. |
| **REQ-M1-FSM-04** | Ao atingir a região da Base Conhecida, o sistema deverá realizar a transição para o estado de aproximação da base conhecida. | Ida à Base Conhecida → Aproximação | Verificar a mudança de estado ao atingir a região definida para a Base Conhecida. |
| **REQ-M1-FSM-05** | Quando as condições de posicionamento forem atendidas, o sistema deverá realizar a transição para o estado de pouso. | Aproximação → Pouso | Verificar a mudança de estado na condição definida. |
| **REQ-M1-FSM-06** |	Após o pouso na base conhecida, o sistema deverá realizar a transição para o estado de nova decolagem. | Pouso → Decolagem | Registrar a confirmação do pouso e verificar a transição para nova decolagem. |
| **REQ-M1-FSM-07** |Após a decolagem da base conhecida, o sistema deverá realizar a transição para o estado de busca e navegação da base desconhecida, mantendo a navegação enquanto o sistema de visão realiza a busca pela Base Desconhecida. | Decolagem → Busca/Navegação Base Desconhecida | Verificar que o estado o Tello permanece em navegação durante a busca visual e que o estado é mantido até a identificação da Base Base Desconhecida. | 
| **REQ-M1-FSM-08** | Após a identificação e localização da base desconhecida pelo sistema de visão, o sistema deverá realizar a transição do estado de busca e navegação para o estado de aproximação da base desconhecida. | Busca/Navegação Base Desconhecida → Aproximação | Fornecer a condição de Base Desconhecida detectada e verificar a transição para o estado de aproximação. |
| **REQ-M1-FSM-09** | Ao chegar à base de decolagem, o sistema deverá realizar a transição para o estado de pouso final. | Retorno → Pouso final | Verificar a mudança de estado. |
| **REQ-M1-FSM-10** | Após a confirmação do pouso final, o sistema deverá entrar em estado de missão concluída. | Pouso final → Finalização | Verificar o encerramento da missão. |

### Fluxo 

`PREPARAÇÃO` → `DECOLAGEM` → `IDA_BASE_CONHECIDA` → `APROXIMAÇÃO` → `POUSO` → `DECOLAGEM` → `BUSCA/NAVEGAÇÃO_BASE_DESCONHECIDA` → `APROXIMAÇÃO` → `POUSO` → `DECOLAGEM` → `RETORNO` → `POUSO_FINAL` → `FINALIZAÇÃO`
---

## 4. Requisitos de Sincronização de Tempo e Comandos (TIME)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M1-TIME-01** | O sistema deverá garantir que os comandos sejam executados na sequência prevista pela lógica da missão. | Toda a missão | Registrar e comparar a ordem dos comandos com a sequência definida. |
| **REQ-M1-TIME-02** | O sistema deverá respeitar os intervalos necessários entre comandos consecutivos enviados ao UAV. | Toda a missão | Executar sequências de comandos e verificar a ausência de perdas relacionadas à temporização. |
| **REQ-M1-TIME-03** | O sistema deverá aguardar a conclusão do pouso antes de iniciar uma nova decolagem. | Pouso → Decolagem | Verificar que a decolagem seguinte só ocorre após confirmação do pouso. |
| **REQ-M1-TIME-04** | O sistema deverá aguardar condições adequadas de aproximação e posicionamento antes de iniciar o pouso. | Aproximação → Pouso | Verificar a ordem dos eventos antes do comando de pouso. |
| **REQ-M1-TIME-05** | O sistema deverá sincronizar os comandos de navegação com os estados definidos pela máquina de estados. | Toda a missão | Comparar registros de comandos e estados. |
| **REQ-M1-TIME-06** | O sistema deverá impedir o envio simultâneo de comandos incompatíveis. | Toda a missão | Simular solicitações concorrentes e verificar o comportamento do sistema. |
