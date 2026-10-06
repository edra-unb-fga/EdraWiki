# Missão 4 — Navegação em Espaço Confinado

## Organização dos requisitos

Os requisitos foram organizados de acordo com a função de cada subsistema dentro da execução da missão e respondem o que o drone faz (1), com que qualidade navega (2), em que sequência faz (3), quais dados usa (4), quando executa cada ação (5) e em qual plataforma isso roda (6).

1. **Controle e Navegação (CTRL):** define as principais ações que o UAV deverá executar ao longo da missão, desde a decolagem até o pouso final.

2. **Malha de Navegação (NAV):** define os requisitos relacionados à trajetória, precisão, correção de desvios e comportamento do UAV durante a navegação no ambiente confinado.

3. **Controle de Estados da Missão (FSM):** define a lógica de sequência da missão e as condições para transição entre suas diferentes etapas.

4. **Câmera e Fluxo de Vídeo (PAYLOAD):** define os requisitos de disponibilização e manutenção do fluxo de vídeo necessário para o sistema de visão computacional.

5. **Sincronização de Tempo e Comandos (TIME):** define os requisitos relacionados à ordem, temporização e sincronização dos comandos enviados ao UAV.

6. **Hardware e Plataforma (HW):** define os requisitos físicos da plataforma DJI Tello e dos componentes adicionais utilizados durante a missão.

---

## 1. Requisitos de Controle e Navegação (CTRL)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M4-CTRL-01** | O UAV deverá realizar a decolagem autonomamente a partir da base de decolagem. | Decolagem | Executar a decolagem sem intervenção humana e verificar o início estável do voo. |
| **REQ-M4-CTRL-02** | O UAV deverá entrar no ambiente confinado pela entrada definida para a missão. | Entrada no labirinto | Verificar se o UAV atravessa corretamente a entrada prevista no layout da missão. |
| **REQ-M4-CTRL-03** | O UAV deverá navegar autonomamente pelo ambiente confinado e obstruído. | Navegação no labirinto | Verificar a execução da navegação sem necessidade de pilotagem manual. |
| **REQ-M4-CTRL-04** | O sistema deverá manter o UAV em condição de voo estável durante a navegação no ambiente confinado. | Navegação no labirinto | Observar o comportamento do UAV durante os testes e verificar a manutenção do voo controlado. |
| **REQ-M4-CTRL-05** | O UAV deverá realizar mudanças de trajetória necessárias à exploração do ambiente. | Exploração do labirinto | Verificar a execução de curvas, mudanças de direção e ajustes de trajetória. |
| **REQ-M4-CTRL-06** | O UAV deverá permanecer em voo durante a exploração do ambiente, exceto quando houver comando de pouso ou encerramento da tentativa. | Navegação no labirinto | Verificar a ausência de pousos não comandados durante os testes. |
| **REQ-M4-CTRL-07** | O sistema de controle deverá permitir o posicionamento do UAV de forma a viabilizar a atuação do sistema de visão. | Busca / Leitura de QR Codes | Verificar se o UAV consegue ajustar ou manter sua posição quando necessário. |
| **REQ-M4-CTRL-08** | Após receber a confirmação de leitura de um QR Code pelo sistema de visão, o UAV deverá prosseguir com a navegação. | Leitura de QR Code → Navegação | Simular ou receber uma confirmação de leitura e verificar a continuação da missão. |
| **REQ-M4-CTRL-09** | O UAV deverá sair do ambiente confinado pela saída definida no regulamento. | Saída do labirinto | Verificar se o UAV atravessa corretamente a saída prevista. |
| **REQ-M4-CTRL-10** | Após sair do ambiente confinado, o UAV deverá deslocar-se autonomamente até a região da base de pouso final. | Saída → Aproximação | Verificar o deslocamento até a região destinada ao pouso. |
| **REQ-M4-CTRL-11** | O UAV deverá realizar o pouso autonomamente na base de pouso final. | Pouso final | Verificar a conclusão do pouso sobre a base prevista. |
| **REQ-M4-CTRL-12** | O sistema deverá permitir o retorno à base de decolagem mediante comando humano, conforme permitido pelo regulamento. | Retorno / Abortagem | Enviar o comando correspondente e verificar a execução do retorno. |

---

## 2. Requisitos de Malha de Navegação (NAV)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M4-NAV-01** | O UAV deverá seguir a trajetória planejada durante a navegação no ambiente confinado. | Navegação no labirinto | Comparar a trajetória planejada com a trajetória executada. |
| **REQ-M4-NAV-02** | O sistema de navegação deverá manter o erro de posição dentro de uma tolerância definida experimentalmente. | Navegação no labirinto | Medir o erro entre a posição de referência e a posição estimada durante os testes. |
| **REQ-M4-NAV-03** | O UAV deverá realizar correções de trajetória quando forem identificados desvios em relação ao percurso previsto. | Navegação no labirinto | Introduzir ou observar desvios e verificar a correção do percurso. |
| **REQ-M4-NAV-04** | O UAV deverá manter margem de segurança em relação às paredes e obstáculos durante a navegação, quando possível. | Navegação no labirinto | Avaliar a distância do UAV em relação aos obstáculos durante os testes. |
| **REQ-M4-NAV-05** | O sistema de navegação deverá permitir mudanças de direção compatíveis com a geometria do labirinto. | Curvas / Mudanças de corredor | Verificar a execução das manobras previstas no percurso. |
| **REQ-M4-NAV-06** | O UAV deverá manter sua posição dentro de uma tolerância definida quando for necessário permanecer temporariamente em um ponto da missão. | Posicionamento / QR Codes | Medir a variação da posição durante o período de permanência. |
| **REQ-M4-NAV-07** | O sistema de navegação deverá conduzir o UAV até a saída correta do ambiente confinado. | Navegação → Saída | Verificar se o percurso executado conduz à saída determinada no regulamento. |

---

## 3. Requisitos de Controle de Estados da Missão (FSM)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M4-FSM-01** | O sistema deverá implementar uma máquina de estados para controlar a sequência de execução da Missão 4. | Toda a missão | Verificar se a execução segue a sequência de estados definida. |
| **REQ-M4-FSM-02** | A máquina de estados deverá possuir um estado inicial de preparação antes da decolagem. | Inicialização | Verificar o estado inicial do sistema antes do voo. |
| **REQ-M4-FSM-03** | O sistema deverá realizar a transição para o estado de decolagem somente após o atendimento das condições necessárias para o início da missão. | Preparação → Decolagem | Verificar se a decolagem só é iniciada após as condições definidas. |
| **REQ-M4-FSM-04** | Após a conclusão da decolagem, o sistema deverá realizar a transição para o estado de entrada no ambiente confinado. | Decolagem → Entrada | Verificar a transição automática entre os estados. |
| **REQ-M4-FSM-05** | Após a entrada no ambiente confinado, o sistema deverá entrar no estado de navegação e exploração. | Entrada → Navegação | Verificar a ativação do estado de navegação. |
| **REQ-M4-FSM-06** | O sistema deverá permitir a transição para um estado de posicionamento quando necessária a atuação do sistema de visão. | Navegação → Posicionamento | Simular uma solicitação e verificar a mudança de estado. |
| **REQ-M4-FSM-07** | Após a conclusão da operação associada ao sistema de visão, o sistema deverá retornar ao estado de navegação. | Posicionamento → Navegação | Verificar o retorno à navegação após a confirmação correspondente. |
| **REQ-M4-FSM-08** | Ao atender à condição de saída do labirinto, o sistema deverá realizar a transição para o estado de saída do ambiente confinado. | Navegação → Saída | Verificar a mudança de estado na condição definida. |
| **REQ-M4-FSM-09** | Após sair do ambiente confinado, o sistema deverá realizar a transição para o estado de aproximação da base de pouso. | Saída → Aproximação | Verificar a ativação do estado de aproximação. |
| **REQ-M4-FSM-10** | Quando as condições de pouso forem atendidas, o sistema deverá realizar a transição para o estado de pouso autônomo. | Aproximação → Pouso | Verificar a mudança de estado ao atingir a região de pouso. |
| **REQ-M4-FSM-11** | Após a confirmação do pouso, o sistema deverá entrar em um estado de missão concluída. | Pouso → Finalização | Verificar o encerramento da execução da missão. |
| **REQ-M4-FSM-12** | O sistema deverá possuir um estado de retorno ou abortagem acionável conforme as condições definidas para a missão. | Estado ativo → Retorno/Abortagem | Acionar a condição correspondente e verificar a transição de estado. |
| **REQ-M4-FSM-13** | A máquina de estados deverá impedir transições incompatíveis com a sequência definida para a missão. | Toda a missão | Tentar executar transições inválidas e verificar seu bloqueio. |

### Fluxo principal sugerido

`PREPARAÇÃO → DECOLAGEM → ENTRADA → NAVEGAÇÃO → POSICIONAMENTO → NAVEGAÇÃO → SAÍDA → APROXIMAÇÃO → POUSO → FINALIZADA`

O estado `RETORNO/ABORTAGEM` poderá ser acessado a partir dos estados de voo quando aplicável.

---

## 4. Requisitos de Controle da Câmera e Fluxo de Vídeo (PAYLOAD)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M4-PAY-01** | O sistema deverá ser capaz de iniciar e manter o fluxo de vídeo da câmera do DJI Tello durante a execução da missão. | Inicialização → Navegação → Pouso | Verificar a recepção contínua do fluxo de vídeo durante um teste da missão. |
| **REQ-M4-PAY-02** | O fluxo de vídeo deverá estar disponível para o módulo de visão durante as etapas de busca e leitura dos QR Codes. | Navegação / Busca por QR Codes | Verificar a disponibilização das imagens para o módulo de visão. |
| **REQ-M4-PAY-03** | O sistema deverá detectar a perda ou interrupção do fluxo de vídeo durante a missão. | Toda a missão | Interromper o fluxo e verificar a identificação da falha. |
| **REQ-M4-PAY-04** | O sistema deverá permitir o restabelecimento do fluxo de vídeo após uma interrupção, quando tecnicamente possível. | Recuperação de falha | Interromper e restabelecer a transmissão e verificar sua retomada. |
| **REQ-M4-PAY-05** | O fluxo de vídeo deverá apresentar latência compatível com sua utilização durante a missão. | Navegação / Posicionamento | Medir a latência entre a aquisição e a disponibilização das imagens. |
| **REQ-M4-PAY-06** | A câmera do DJI Tello deverá permanecer operacional durante o período necessário para a execução da missão. | Toda a missão | Executar um teste com duração compatível com uma tentativa e verificar a disponibilidade da câmera. |
| **REQ-M4-PAY-07** | O sistema de iluminação auxiliar deverá fornecer condições adequadas para a aquisição de imagens em ambiente escuro. | Navegação / Busca por QR Codes | Realizar testes com baixa iluminação e verificar a aquisição de imagens com o LED ativo. |
| **REQ-M4-PAY-08** | A transmissão do fluxo de vídeo não deverá impedir o envio dos comandos necessários ao controle do DJI Tello. | Toda a missão | Executar transmissão de vídeo e comandos simultaneamente e verificar o funcionamento do sistema. |

---

## 5. Requisitos de Sincronização de Tempo e Comandos (TIME)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M4-TIME-01** | O sistema deverá garantir que os comandos enviados ao DJI Tello sejam executados na sequência prevista pela lógica da missão. | Toda a missão | Registrar os comandos enviados e comparar sua ordem com a sequência prevista. |
| **REQ-M4-TIME-02** | O sistema deverá respeitar os intervalos necessários entre comandos consecutivos enviados ao DJI Tello. | Toda a missão | Executar sequências de comandos e verificar a ausência de perda ou rejeição devido à temporização. |
| **REQ-M4-TIME-03** | O sistema deverá aguardar a conclusão ou confirmação de uma ação antes de executar uma ação que dependa dela, quando aplicável. | Transições entre etapas | Verificar se uma ação dependente não é iniciada prematuramente. |
| **REQ-M4-TIME-04** | O sistema deverá sincronizar o envio de comandos com os estados definidos pela máquina de estados da missão. | Toda a missão | Comparar os registros de estados e comandos. |
| **REQ-M4-TIME-05** | O sistema deverá evitar o envio de comandos conflitantes ao UAV. | Toda a missão | Simular solicitações concorrentes e verificar a priorização adequada dos comandos. |
| **REQ-M4-TIME-06** | O sistema deverá controlar o tempo de permanência nos estados que exijam espera ou posicionamento. | Posicionamento / QR Codes | Medir o tempo de permanência e verificar as condições utilizadas para transição. |
| **REQ-M4-TIME-07** | O sistema deverá monitorar o tempo total da tentativa, considerando o limite máximo de 10 minutos definido no regulamento. | Toda a missão | Verificar o acompanhamento do tempo desde o início até o encerramento da tentativa. |
| **REQ-M4-TIME-08** | O sistema deverá permitir a definição de tempos máximos de espera para ações da missão. | Navegação / Posicionamento / Comunicação | Simular uma ação sem conclusão e verificar a identificação do tempo excedido. |
| **REQ-M4-TIME-09** | Caso o tempo máximo definido para uma ação seja excedido, o sistema deverá executar o comportamento de recuperação ou abortagem definido para a situação. | Tratamento de falhas | Forçar uma condição de timeout e verificar a resposta do sistema. |
| **REQ-M4-TIME-10** | O sistema deverá registrar informações temporais suficientes para ordenar os principais eventos, comandos e mudanças de estado da missão. | Toda a missão | Analisar os registros da execução e verificar a sequência temporal dos eventos. |

---

## 6. Requisitos de Hardware e Plataforma (HW)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M4-HW-01** | A Missão 4 deverá utilizar o **DJI Tello** como plataforma física de desenvolvimento e testes. | Toda a missão | Verificar a utilização do DJI Tello nos testes físicos. |
| **REQ-M4-HW-02** | O conjunto do UAV e dos componentes adicionados deverá atender ao limite máximo de **330 mm**, conforme estabelecido pelo regulamento. | Preparação / Pré-voo | Medir as dimensões finais do conjunto. |
| **REQ-M4-HW-03** | O DJI Tello deverá utilizar **protetores de hélice** durante a execução da missão. | Preparação / Pré-voo | Realizar inspeção visual antes do voo. |
| **REQ-M4-HW-04** | A plataforma deverá permitir o envio dos comandos necessários para decolagem, navegação, posicionamento e pouso. | Decolagem → Navegação → Pouso | Executar os comandos e verificar a resposta do DJI Tello. |
| **REQ-M4-HW-05** | O sistema deverá ser capaz de obter do DJI Tello os dados necessários à execução e ao monitoramento da missão. | Toda a missão | Verificar a recepção dos dados utilizados pelo sistema. |
| **REQ-M4-HW-06** | A plataforma deverá disponibilizar o fluxo de imagem da câmera do DJI Tello para utilização pelo sistema de visão. | Exploração / QR Codes | Verificar a recepção do fluxo de vídeo durante o voo. |
| **REQ-M4-HW-07** | O UAV deverá possuir um **sistema auxiliar de iluminação por LED** capaz de iluminar a região à frente do drone durante a navegação no ambiente escuro. | Entrada / Navegação no labirinto | Realizar teste em baixa iluminação e verificar a área iluminada à frente do UAV. |
| **REQ-M4-HW-08** | O sistema de iluminação deverá ser instalado de forma a não comprometer a estabilidade, a movimentação, os protetores de hélice ou a operação segura do DJI Tello. | Toda a navegação | Realizar testes de voo com o sistema de iluminação instalado. |
| **REQ-M4-HW-09** | O sistema de iluminação deverá possuir alimentação suficiente para permanecer operacional durante o período necessário à execução da missão. | Navegação no labirinto | Testar o funcionamento contínuo da iluminação durante o período previsto. |
| **REQ-M4-HW-10** | A plataforma DJI Tello, com os componentes adicionais instalados, deverá ser validada quanto à capacidade de operar de forma estável e segura no ambiente confinado previsto para a Missão 4. | Validação / Navegação | Realizar testes progressivos em ambiente confinado e avaliar estabilidade, controle e segurança do voo. |
