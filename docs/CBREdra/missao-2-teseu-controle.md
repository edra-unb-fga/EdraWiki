**Equipe TESEU**

# **RELATÓRIO DE REQUISITOS DO SISTEMA**

**FASE 2 — TRANSPORTE DE PACOTES**

Síntese SRR \+ DRD \+ Verificação e Validação (V\&V)

Documento destinado à apresentação técnica e à organização interna da equipe.

Brasília — DF

2026

1 INTRODUÇÃO

Este documento consolida os requisitos de sistema, a arquitetura de referência e o plano de verificação e validação para a Fase 2 — Transporte de Pacotes. A missão consiste em identificar, coletar, transportar e entregar um kit de primeiros socorros entre duas bases conhecidas, com possibilidade de retorno autônomo à base de origem. A estrutura adota rastreabilidade entre requisito, solução e teste, seguindo a prática de engenharia de requisitos da ISO/IEC/IEEE 29148 e de verificação e validação da IEEE 1012\.

1.1 Escopo

O sistema compreende aeronave multirrotor, propulsão e alimentação, flight controller, navegação, computador de bordo, percepção visual, mecanismo de manipulação, software de autonomia e estação de supervisão. O kit não deverá receber marcadores, placa magnética ou cabo suspenso.

1.2 Premissas e restrições da missão

| ID | Condição |
| ----- | ----- |
| EXT-01 | 2 bases: origem/coleta (A) e destino/entrega (B). |
| EXT-02 | 1 kit de primeiros socorros; massa mínima de 160 g. |
| EXT-03 | Kit pode variar de posição/orientação dentro da área de coleta. |
| EXT-04 | Na coleta e entrega, o trem de pouso deve permanecer integralmente dentro da base. |
| EXT-05 | Sem QR Code, ArUco, AprilTag ou outro marcador no kit. |
| EXT-06 | Sem cabo suspenso e sem placa magnética no kit. |
| EXT-07 | Até 3 tentativas; 30 min totais; máximo de 10 min por tentativa. |
| EXT-08 | Retorno à origem pode ser manual; retorno autônomo é desejável para maximizar a pontuação. |
| EXT-09 | Pouso fora de uma base durante coleta/entrega encerra a tentativa. |

1.3 Objetivo de engenharia

Maximizar a probabilidade de completar o ciclo coleta → entrega → retorno, priorizando confiabilidade de pouso e manipulação sobre velocidade. A solução deverá permitir repetibilidade e diagnóstico pós-voo.

2 SYSTEM REQUIREMENTS REVIEW (SRR)

O SRR estabelece o que o sistema deve fazer, independentemente da tecnologia escolhida. Requisitos P0 são críticos para a missão; P1 são importantes para desempenho, manutenção ou pontuação adicional.

| ID | Requisito | Pri. | V |
| ----- | ----- | ----- | ----- |
| SYS-01 | Executar autonomamente a missão de transporte A → B. | P0 | D |
| SYS-02 | Identificar o kit sem marcador artificial. | P0 | T |
| SYS-03 | Realizar pouso válido na área de coleta. | P0 | T |
| SYS-04 | Capturar o kit sem cabo e sem elemento magnético instalado no kit. | P0 | T |
| SYS-05 | Confirmar que o kit foi capturado e levantado antes do transporte. | P0 | T |
| SYS-06 | Transportar o kit até B sem soltura não comandada. | P0 | T |
| SYS-07 | Realizar pouso válido e liberar o kit em B. | P0 | T |
| SYS-08 | Executar retorno autônomo B → A após entrega. | P1 | D |
| SYS-09 | Interromper descida quando percepção, posicionamento ou altura forem inadequados. | P0 | T |
| SYS-10 | Possuir failsafes para perda de comunicação, bateria e estimativa de estado. | P0 | T |
| SYS-11 | Respeitar o limite de 10 min por tentativa. | P0 | A/T |
| SYS-12 | Registrar estados, eventos e parâmetros críticos da missão. | P1 | I/T |
| SYS-13 | Operar com a massa máxima de carga definida pela equipe. | P0 | T |
| SYS-14 | Permitir abortamento controlado da missão. | P0 | D |

Legenda de verificação: I \= inspeção; A \= análise; T \= teste; D \= demonstração.

3 DESIGN REQUIREMENTS DOCUMENT (DRD)

As escolhas abaixo são uma baseline de projeto. Elas podem ser substituídas por soluções equivalentes, desde que atendam aos requisitos do SRR.

| Subsistema | Baseline | Função |
| ----- | ----- | ----- |
| Voo | Multirrotor \+ Pixhawk/PX4 | Estabilização, estimação e controle de baixo nível. |
| Computação | Companion computer | Percepção, lógica de missão e supervisão. |
| Middleware | ROS 2 \+ interface PX4/DDS | Integração entre autonomia e flight controller. |
| Visão | Câmera inferior \+ OpenCV \+ detector de objetos | Identificação e localização relativa do kit. |
| Altura | Rangefinder/ToF/LiDAR | Medição de distância ao solo na aproximação final. |
| Manipulação | Garra mecânica | Captura, retenção e liberação da carga. |
| Supervisão | QGroundControl/GCS | Monitoramento, configuração e intervenção. |
| Simulação | PX4 SITL \+ Gazebo | Validação da integração antes do voo. |

3.1 Visão e pouso

A visão deverá detectar o kit por suas características visuais, sem depender de marcadores. Um detector como YOLO é uma opção de implementação, não um requisito. A posição do objeto na imagem deverá ser combinada com calibração da câmera, atitude da aeronave e medição de altura para estimar o erro relativo. A descida final somente deverá ser habilitada quando o alinhamento estiver dentro da margem definida pela equipe e a geometria do trem de pouso permanecer compatível com a área da base.

3.2 Manipulação

A garra deverá possuir retenção mecânica suficiente para a massa máxima definida. “Garra fechada” não será considerada equivalente a “carga capturada”: o sistema deverá confirmar a captura por sensor de posição, força/carga ou combinação equivalente antes do deslocamento horizontal.

3.3 Arquitetura de software

A lógica de missão deverá ser implementada como máquina de estados ou árvore de comportamento. O fluxo mínimo é: INIT → PREFLIGHT → TAKEOFF → TRANSIT\_TO\_A → SEARCH\_KIT → ALIGN\_PICKUP → LAND\_PICKUP → GRIP → VERIFY\_GRIP → LIFT\_VERIFY → TRANSIT\_TO\_B → ALIGN\_DELIVERY → LAND\_DELIVERY → RELEASE → AUTONOMOUS\_RETURN → LAND\_HOME → COMPLETE. Falhas críticas devem possuir transições explícitas para HOLD, ABORT ou RETURN.

4 REQUISITOS DE INTERFACE E SEGURANÇA

| ID | Requisito | Critério resumido |
| ----- | ----- | ----- |
| INT-01 | ROS 2/PX4 deve possuir interface definida e versionada. | Build reproduzível. |
| INT-02 | Flight controller permanece responsável pelo controle de baixo nível. | Sem comando direto de motores pelo ROS 2\. |
| INT-03 | Heartbeat/Offboard e failsafe devem ser configurados e testados. | Perda do companion gera ação segura. |
| SAF-01 | Perda visual na aproximação bloqueia a descida. | Teste reproduzível. |
| SAF-02 | Bateria insuficiente bloqueia avanço para a etapa incompatível com retorno. | Teste com limiar configurado. |
| SAF-03 | Perda da confirmação da carga impede o transporte. | FSM permanece em estado seguro. |
| SAF-04 | Limites de velocidade, altitude, área e tempo devem ser configuráveis. | Parâmetros documentados. |

5 VERIFICAÇÃO E VALIDAÇÃO (V\&V)

A V\&V será progressiva: simulação → bancada → voo sem carga → voo com carga → missão completa. O Gazebo valida principalmente integração, navegação, percepção e lógica; requisitos mecânicos e de interação real com a carga exigem ensaios físicos.

5.1 Campanha mínima no Gazebo

| ID | Ensaio | Critério de aprovação |
| ----- | ----- | ----- |
| GZ-01 | Inicialização PX4 SITL \+ Gazebo \+ ROS 2 | Todos os nós, sensores e interfaces ativos. |
| GZ-02 | Decolagem e navegação A→B | Trajetória executada sem instabilidade. |
| GZ-03 | Detecção do kit em posições/orientações variadas | Detecção válida acima do limiar definido. |
| GZ-04 | Controle visual de alinhamento | Erro reduz de forma estável e sem oscilação excessiva. |
| GZ-05 | Pouso na Base A | Geometria do trem de pouso permanece na região válida. |
| GZ-06 | Perda visual durante descida | Descida é interrompida e sistema entra em recuperação. |
| GZ-07 | Falso positivo | Objetos semelhantes não validam o kit. |
| GZ-08 | Falha do rangefinder | Descida final é bloqueada. |
| GZ-09 | Perda de Offboard/companion | PX4 executa failsafe configurado. |
| GZ-10 | Bateria baixa | Missão entra em retorno/abort conforme configuração. |
| GZ-11 | Captura confirmada | FSM só avança após condição de carga atendida. |
| GZ-12 | Falha de captura | Transporte não é iniciado. |
| GZ-13 | Entrega em B | Pouso válido \+ liberação confirmada. |
| GZ-14 | Retorno B→A | Retorno autônomo e pouso final. |
| GZ-15 | Missão completa | Ciclo integral executado sem intervenção humana. |

5.2 Testes de robustez no Gazebo

| Teste | Perturbação | Objetivo |
| ----- | ----- | ----- |
| GZ-R01 | Ruído na estimativa de posição | Avaliar tolerância do controle. |
| GZ-R02 | Variação de iluminação/contraste | Avaliar robustez da visão. |
| GZ-R03 | Oclusão parcial do kit | Validar rastreamento e confirmação temporal. |
| GZ-R04 | Atraso de processamento de imagem | Verificar estabilidade e limitar oscilações. |
| GZ-R05 | Variação de atitude na aproximação | Verificar erro relativo e segurança da descida. |
| GZ-R06 | Falha intermitente de sensor | Verificar transição para estado seguro. |

5.3 Matriz V\&V resumida

| Requisito | Teste principal | Ambiente | Evidência |
| ----- | ----- | ----- | ----- |
| SYS-02 | GZ-03 | Gazebo \+ vídeo real | Métricas e logs de detecção |
| SYS-03 | GZ-05 | Gazebo \+ voo | Log de posição \+ vídeo |
| SYS-04 | GZ-11 | Gazebo \+ bancada | Estado da garra \+ ensaio |
| SYS-05 | GZ-11/GZ-12 | Gazebo \+ bancada | Evento VERIFY\_GRIP |
| SYS-06 | GZ-15 | Gazebo \+ voo | Log de missão |
| SYS-07 | GZ-13 | Gazebo \+ voo | Pouso \+ evento RELEASE |
| SYS-08 | GZ-14 | Gazebo \+ voo | Trajetória e pouso A |
| SYS-09 | GZ-06/GZ-08 | Gazebo | Mudança de estado |
| SYS-10 | GZ-09/GZ-10 | Gazebo \+ voo | Failsafe registrado |
| SYS-11 | GZ-15 | Gazebo | Timestamp de início/fim |
| SYS-13 | Teste de massa | Bancada \+ voo | Massa e desempenho |
| SYS-14 | Teste de abort | Gazebo \+ voo | Comando \+ estado final |

6 CRITÉRIO DE PRONTIDÃO

A missão será considerada pronta para competição quando: (i) todos os requisitos P0 estiverem validados; (ii) o ciclo coleta → entrega → retorno estiver funcional; (iii) os testes Gazebo GZ-01 a GZ-15 estiverem concluídos; (iv) os principais modos de falha tiverem sido ensaiados; e (v) a configuração final de software e parâmetros estiver versionada e reproduzível.

6.1 Riscos prioritários

| Risco | Impacto | Mitigação principal |
| ----- | ----- | ----- |
| Falha de detecção | Alto | Dataset real \+ confirmação temporal \+ fallback seguro. |
| Pouso fora da área | Crítico | Geometria do trem de pouso \+ margem \+ velocidade final baixa. |
| Falha de captura | Crítico | Garra mecânica \+ sensor de confirmação \+ testes de bancada. |
| Queda da carga | Crítico | Retenção positiva \+ verificação após levantamento. |
| Perda do companion | Crítico | Failsafe no PX4 \+ watchdog. |
| Bateria insuficiente | Crítico | Reserva energética \+ monitoramento \+ missão limitada. |
| Latência visual | Alto | Pipeline assíncrono \+ controle limitado \+ teste GZ-R04. |

7 RASTREABILIDADE E CONTROLE DE CONFIGURAÇÃO

Cada requisito P0/P1 deverá possuir pelo menos um teste associado e um registro de resultado. Cada ensaio deverá registrar versão do firmware PX4, versão do pacote de mensagens/ROS 2, versão do modelo de visão, parâmetros de voo, configuração da câmera, versão da garra, massa da aeronave/carga, data e resultado. Alterações deverão ser rastreadas em controle de versão.

8 REFERÊNCIAS

IEEE. ISO/IEC/IEEE 29148:2018: Systems and software engineering — Life cycle processes — Requirements engineering. New York: IEEE, 2018\.

IEEE. IEEE 1012-2024: Standard for System, Software, and Hardware Verification and Validation. New York: IEEE, 2024\.

PX4. ROS 2 User Guide. PX4 User Guide, 2026\. Disponível em: \<https\://docs.px4.io/main/en/ros2/user\_guide\>. Acesso em: 6 out. 2026\.

PX4. Simulation. PX4 User Guide, 2026\. Disponível em: \<https\://docs.px4.io/main/en/simulation/\>. Acesso em: 6 out. 2026\.

\[COMPETIÇÃO\]. Enunciado da Fase 2 — Transporte de Pacotes. Documento fornecido à equipe. 2026\.