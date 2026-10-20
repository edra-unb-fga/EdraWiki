# Missão 3 — Interação Humano-Robô

## Organização dos requisitos

Os requisitos foram organizados de acordo com a função de cada subsistema dentro da execução da missão e respondem o que o drone faz (1), com que qualidade e resposta navega sob comandos visuais (2), em que sequência lógica opera (3), quais dados e fluxos de câmera utiliza (4), e como sincroniza temporização e comandos críticos (5). Esta missão considera o DJI Tello como plataforma física da Missão 3.

1. **Controle e Navegação (CTRL):** define as ações de controle de voo que o UAV deverá executar ao longo da missão, desde a decolagem e enquadramento até os pousos nas bases e o retorno final.

2. **Malha de Navegação e Interação (NAV):** define os requisitos de estabilidade, comportamento dinâmico sob comandos de gestos, tolerância de erros e regras de não rastreamento.

3. **Controle de Estados da Missão (FSM):** define a lógica sequencial da missão, transições entre etapas, histórico de bases visitadas e condições de encerramento.

4. **Câmera e Fluxo de Vídeo (PAYLOAD):** define os requisitos de disponibilização, integridade e transmissão do fluxo de vídeo necessário para a detecção do operador e comandos visuais.

5. **Sincronização de Tempo e Comandos (TIME):** define a ordem, temporização, latência e cadência de envio de comandos para o UAV e publicação no terminal da Banca.

---

## 1. Requisitos de Controle e Navegação (CTRL)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M3-CTRL-01** | O UAV deverá realizar a decolagem autonomamente a partir da base de decolagem mediante comando do computador. | Decolagem | Executar a decolagem via software sem intervenção manual e verificar a subida estável. |
| **REQ-M3-CTRL-02** | O UAV deverá realizar aproximação autônoma até o operador humano localizado no centro da arena. | Decolagem → Aproximação | Observar o deslocamento autônomo do UAV em direção ao operador logo após estabilizar o voo inicial. |
| **REQ-M3-CTRL-03** | O UAV deverá ajustar sua orientação e altitude para enquadrar o operador humano no campo de visão ótimo. | Aproximação / Enquadramento | Verificar o enquadramento do operador no feed de vídeo antes do início das ordens de navegação. |
| **REQ-M3-CTRL-04** | O UAV deverá movimentar-se na arena guiado exclusivamente por comandos visuais decodificados a partir de gestos/ações do operador. | Navegação guiada | Enviar gestos de translação/rotação e verificar a execução correspondente pelo drone. |
| **REQ-M3-CTRL-05** | O UAV não deverá utilizar trajetórias pré-programadas nem detecção de marcadores visuais para se aproximar ou pousar nas bases. | Navegação guiada | Garantir que o drone não navegue por waypoints prévios e mantenha os marcadores visuais das bases desativados/ocultos. |
| **REQ-M3-CTRL-06** | O UAV deverá executar a rotina de pouso sobre a base de pouso selecionada mediante comando visual correspondente. | Pouso em base | Enviar o comando gestual de pouso e verificar o assentamento completo do trem de pouso sobre a base com motores desligados. |
| **REQ-M3-CTRL-07** | O UAV deverá realizar nova decolagem a partir da base de pouso para continuar a missão rumo às próximas bases. | Decolagem da base | Acionar o comando de prosseguimento e verificar a decolagem estável a partir da base visitada. |
| **REQ-M3-CTRL-08** | O UAV deverá retornar autonomamente à base de decolagem ao finalizar a visitação das bases de pouso. | Retorno à base | Verificar a navegação autônoma de regresso e pouso na base de decolagem sem pilotagem manual. |
| **REQ-M3-CTRL-09** | O sistema deverá permitir a interrupção imediata ou encerramento da tentativa caso o UAV pouse fora de uma base ou a segurança seja violada. | Emergência / Falha | Disparar o comando de interrupção e verificar o corte ou parada de emergência. |

---

## 2. Requisitos de Malha de Navegação e Interação (NAV)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M3-NAV-01** | O sistema deverá manter o UAV em condição de voo pairado (*hover*) estável a uma altura mínima de 1 metro na ausência de novos comandos de gesto válidos. | Navegação guiada / Espera | Suspender o envio de gestos e verificar a sustentação estática estável do drone. |
| **REQ-M3-NAV-02** | O sistema de controle não deverá utilizar algoritmos de seguimento automático contínuo do operador (*detect-and-follow*). | Toda a missão | Verificar se os deslocamentos ocorrem estritamente por comandos discretos de controle derivados de gestos e não por perseguição visual. |
| **REQ-M3-NAV-03** | O sistema deverá limitar as velocidades máximas de translação e rotação dentro de limites seguros para voo próximo a humanos. | Navegação guiada | Medir as velocidades de resposta aos comandos gestuais e verificar o cumprimento dos limites configurados no software. |
| **REQ-M3-NAV-04** | O UAV deverá suavizar variações bruscas de velocidade ao alternar entre diferentes comandos de gestos. | Navegação guiada | Aplicar comandos gestuais alternados e observar a transição contínua sem oscilações excessivas no Tello. |
| **REQ-M3-NAV-05** | O sistema deverá cessar movimentações e entrar em modo de sustentação de segurança caso perca a detecção visual do operador por tempo superior ao limite tolerado. | Perda visual transitória | Simular obstrução do operador e verificar a parada imediata de translação do UAV. |

---

## 3. Requisitos de Controle de Estados da Missão (FSM)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M3-FSM-01** | O sistema deverá implementar uma máquina de estados finitos (FSM) para orquestrar toda a sequência da Missão 3. | Toda a missão | Monitorar a transição lógica dos estados ao longo da execução da tentativa. |
| **REQ-M3-FSM-02** | A FSM deverá possuir um estado inicial de preparação e verificação de prontidão dos tópicos de comunicação antes da decolagem. | Preparação | Verificar checagem de conexão e telemetria antes de liberar o comando de decolagem. |
| **REQ-M3-FSM-03** | A FSM deverá transicionar para o estado de aproximação logo após a confirmação da decolagem autônoma bem-sucedida. | Decolagem → Aproximação | Checar mudança de estado assim que a altitude de cruzeiro inicial for atingida. |
| **REQ-M3-FSM-04** | A FSM deverá transicionar para o estado de reconhecimento do operador e exigir confirmação visual antes de autorizar comandos de voo. | Aproximação → Reconhecimento | Verificar que ordens de navegação permanecem bloqueadas até a validação da detecção do operador. |
| **REQ-M3-FSM-05** | A FSM deverá transicionar para o estado de navegação interativa quando o operador estiver ativamente enviando comandos válidos. | Reconhecimento → Navegação | Emitir gesto válido e verificar a mudança de estado para execução de movimento. |
| **REQ-M3-FSM-06** | A FSM deverá comutar para o estado de pouso na base ao receber a instrução gestual de pouso sobre uma plataforma. | Navegação → Pouso em Base | Enviar gesto de pouso e verificar transição para descida e corte de motores. |
| **REQ-M3-FSM-07** | A FSM deverá manter registro das bases já visitadas para mitigar repetições desnecessárias e penalidades de pontuação. | Pouso em Base | Registrar identificador da base visitada e atualizar a contagem de alvos cumpridos. |
| **REQ-M3-FSM-08** | Após o pouso na base, a FSM deverá permitir transição para decolagem intermediária rumo à próxima base. | Pouso em Base → Nova Decolagem | Comandar nova decolagem a partir da base e retornar ao estado de navegação interativa. |
| **REQ-M3-FSM-09** | Ao atingir a totalidade das bases previstas ou receber comando de encerramento, a FSM deverá entrar no estado de retorno autônomo à base de decolagem. | Navegação / Pouso → Retorno | Disparar encerramento da rota e checar início da navegação de regresso. |
| **REQ-M3-FSM-10** | Ao pousar na base de decolagem, a FSM deverá transicionar para o estado de missão concluída com corte completo de atuadores. | Pouso final → Finalização | Confirmar pouso final, desligamento dos motores e parada dos nós de controle. |
| **REQ-M3-FSM-11** | A FSM deverá possuir um estado de abortagem/failsafe acessível a partir de qualquer estado de voo. | Estado ativo → Abortagem | Simular condição de emergência ou comando de rádio e verificar interrupção imediata da FSM. |

### Fluxo principal sugerido

`PREPARAÇÃO → DECOLAGEM → APROXIMAÇÃO → RECONHECIMENTO_HUMANO → NAVEGAÇÃO_INTERATIVA ⇄ POUSO_BASE → RETORNO_BASE_DECOLAGEM → POUSO_FINAL → FINALIZADA`

O estado `ABORTAGEM/FAILSAFE` poderá ser acessado a partir de qualquer estado de voo mediante perda crítica ou comando de emergência.

---

## 4. Requisitos de Controle da Câmera e Fluxo de Vídeo (PAYLOAD)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M3-PAY-01** | O sistema deverá iniciar e manter a recepção contínua do fluxo de vídeo da câmera frontal do DJI Tello desde a inicialização. | Toda a missão | Verificar a chegada ininterrupta dos frames de vídeo na estação de controle durante o voo. |
| **REQ-M3-PAY-02** | O fluxo de vídeo deverá estar disponível para o nó de visão computacional para detecção de pose/esqueleto do operador. | Aproximação → Navegação interativa | Checar a disponibilização das imagens para o extrator de gestos em tempo real. |
| **REQ-M3-PAY-03** | O sistema deverá publicar no terminal os dados visuais do operador (coordenadas no mundo, esqueleto ou detecção) para auditoria da Banca. | Reconhecimento / Navegação | Verificar exibição contínua dos dados corporais do operador no terminal do computador. |
| **REQ-M3-PAY-04** | O sistema deverá publicar no terminal o nome de cada comando visual reconhecido em tempo real. | Navegação interativa | Realizar gestos e constatar a impressão imediata dos respectivos nomes no terminal. |
| **REQ-M3-PAY-05** | O sistema de controle deverá identificar falhas, congelamento ou interrupção do fluxo de vídeo. | Toda a missão | Interromper a transmissão de vídeo e constatar a detecção da falha pelo supervisor de voo. |
| **REQ-M3-PAY-06** | A transmissão e processamento do vídeo não deverão degradar a largura de banda necessária para o envio dos pacotes de controle UDP. | Toda a missão | Validar envio simultâneo de comandos e recepção de vídeo sem perdas de controle. |

---

## 5. Requisitos de Sincronização de Tempo e Comandos (TIME)

| ID | Requisito | Fluxo da missão | Validação |
|---|---|---|---|
| **REQ-M3-TIME-01** | O sistema deverá processar comandos gestuais a uma taxa temporal compatível com a estabilidade de controle do Tello. | Navegação interativa | Medir a taxa de atualização dos comandos de velocidade (ideal entre 10 Hz e 20 Hz) e verificar comportamento suave. |
| **REQ-M3-TIME-02** | O sistema deverá implementar persistência temporal mínima para validação de comandos gestuais antes de comutar movimentos. | Reconhecimento → Navegação | Testar gestos rápidos e verificar se apenas comandos sustentados por janela de tempo definida são repassados ao UAV. |
| **REQ-M3-TIME-03** | O sistema deverá garantir a ordem sequencial estrita dos comandos enviados ao Tello, descartando instruções obsoletas recebidas fora de ordem. | Toda a missão | Injetar comandos em rajada e verificar que o atuador executa somente o pacote mais recente. |
| **REQ-M3-TIME-04** | O sistema deverá respeitar a janela de tempo total da tentativa estipulada em regulamento (até 15 minutos por tentativa dentro dos 30 minutos totais). | Toda a missão | Monitorar o temporizador de missão e acionar procedimentos de retorno ou pouso preventivo antes do encerramento do tempo regulamentar. |