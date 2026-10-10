# [EDRA] Especificação e Requisitos de Visão Computacional (CV): Fase 1

> **Escopo:** levantamento de requisitos do subsistema de Visão Computacional para a **Fase 1: Localização e Mapeamento** do **Desafio Interno da EDRA**. Os requisitos, as métricas e os testes são dimensionados para o Interno. As regras da Flying Robot League da CBR 2026 entram só como referência: para comparar e para preencher o que o edital interno deixa em branco.

> **Pendências do documento:**
> * **Métricas:** os valores dos requisitos (FPS, cm, %, tamanho do dataset) são propostas iniciais.
> * **Hardware do Teseu:** **(ESPERANDO RETORNO DAS SPECS)**. Até lá, o texto assume câmera inferior RGB e Raspberry Pi, e os cálculos de campo de visão usam a Pi Camera v2 (62° × 49°) como exemplo.

---

### 1. IDENTIFICAÇÃO DO PROJETO

* **Fase(s) da Competição:**
  * [x] **Fase 1: Mapeamento e Detecção de Bases**
  * [ ] **Fase 2: Detecção e Rastreamento de Kit de Primeiros Socorros**
  * [ ] **Fase 3: Interação Humano-Robô (Reconhecimento de Gestos/Operador)**
  * [ ] **Fase 4: Leitura de QR Codes em Ambientes Confinados/Escuros**
* **Plataforma e Câmera(s):**
  * [x] **X500 / Teseu (Fases 1, 2, 3)**: *Câmera(s): Câmera Inferior RGB, fixa no frame, apontada para baixo **(ESPERANDO RETORNO DAS SPECS)***
  * [ ] **Tello / Mini UAV (Fases 1, 3, 4)**: *alternativa analisada na seção 2.5*
* **Responsáveis da Visão Computacional:** Enzo Emir
* **Data:** 08/10/2026

---

### 2. ENTENDIMENTO E DESAFIO VISUAL DA MISSÃO

#### 2.1 Descrição do Desafio de Visão

O drone decola da base de decolagem, percorre a arena de 8 m × 8 m, pousa **uma vez** em cada uma das **2 bases de pouso** e retorna à base de decolagem. A posição de uma base é informada antes, a outra é colocada aleatoriamente pela Banca e precisa ser encontrada durante o voo. Sem GPS e sem beacons, a câmera é a única fonte para encontrar a base e alinhar o pouso.

O que a câmera precisa enxergar:

* **Base de pouso (alvo):** quadrado azul de 1 m × 1 m, círculo amarelo de 0,8 m de diâmetro e cruz amarela de 0,5 m. Linhas amarelas de 5 cm (largura informada só na CBR, creio que será igual).
* **Base de decolagem (alvo só no final):** retângulo azul de 2 m × 1,5 m, a ~0,5 m de altura, com retângulo amarelo de 1,5 m × 1 m e círculo azul de 0,8 m de diâmetro, com cruz amarela de 0,5 m. Pousar nela encerra a tentativa, então não pode ser confundida com base de pouso. No fim, o pouso autônomo nela dobra a pontuação, então a visão também precisa centralizar nela.
* **Centro da base:** o pouso só vale com **todo o trem de pouso** apoiado na base.

O que torna o problema difícil:

* Bases entre **0 e 1,5 m** de altura: o tamanho aparente depende da distância câmera–base, não da altitude do drone em relação ao chão.
* Mesmo a base de posição informada precisa de visão para o pouso preciso, porque a odometria sem GPS acumula erro.
* Pouso repetido custa 5 pontos e pouso fora de base encerra a tentativa: falso positivo é caro.
* Depois de anunciada a primeira tentativa oficial, **código e parâmetros ficam congelados**.

#### 2.2 Fluxo Operacional de Processamento (Passo a Passo)

1. **Captura do Frame:** câmera inferior RGB, 640 × 480 a 30 FPS, exposição automática ligada.
2. **Pré-processamento:** correção de distorção com os intrínsecos da calibração. Redimensionamento para 320 × 320 (entrada do detector). Na etapa clássica, conversão para HSV e CLAHE no canal V, só dentro da região detectada.
3. **Detecção / Inferência:**
   * **Etapa 1 (YOLO11n):** localiza as bases e classifica em `base_pouso` ou `base_decolagem`.
   * **Etapa 2 (OpenCV):** dentro da caixa detectada, segmenta o amarelo, ajusta uma elipse ao círculo e localiza a cruz. O centro da elipse/cruz é o centro da base, e os braços da cruz dão a orientação. Na base de decolagem o círculo é azul sobre amarelo, então o refino usa a cruz e a borda do círculo azul.
   * **Distância:** com o tamanho real conhecido (círculo de 0,8 m, cruz de 0,5 m), o tamanho em pixels e a distância focal dão a distância até a base, seja qual for a altura dela.
4. **Pós-processamento e Saída:**
   * Confirmação temporal: alvo confirmado ao aparecer em 5 dos últimos 8 frames.
   * Suavização do centro (Kalman ou média móvel).
   * Conversão de pixels para metros: `dx`, `dy` (m, no referencial do drone) e `dyaw` (rad).
   * Publicação do erro, da flag de detecção e da classe.

   Decidir se uma base "já foi visitada" é papel do nó de missão/navegação, que associa cada detecção à posição estimada do drone. A visão entrega detecções estáveis e separadas.

#### 2.3 Restrições e Desafios de Imagem na Arena

* **Marcadores:** o desenho da própria base é o marcador. Não há ArUco/QR nas bases da Fase 1. Proibido GPS/RTK e sistema externo de localização.
* **Condições de Iluminação:** luz solar por janelas (iluminação não uniforme, sombras duras). Piso liso e homogêneo com possível reflexo. Cobertura de MDF de **qualquer cor**, ou seja, fundo imprevisível.
* **Perspectiva e Oclusão:**
  * **(ESPERANDO RETORNO DAS SPECS)** Usando a Pi Camera v2 como exemplo: a 3 m do chão, o quadro cobre ~3,6 m × 2,7 m no piso, mas só ~1,8 m × 1,4 m sobre uma base a 1,5 m de altura.
  * Inclinação do drone (roll/pitch) desloca e deforma o alvo, já que a câmera é fixa.

#### 2.4 Levantamento do Edital: números, obrigações e proibições

A coluna "Interno" é a que vale para este documento, e a coluna "CBR" está só para comparação. Seções do Interno são citadas pelo número e as da CBR pelo título (o documento da CBR não é numerado).

| Item do edital | Interno | CBR | Fonte | O que muda para a visão |
| :--- | :--- | :--- | :--- | :--- |
| Arena | 8 m × 8 m | 8 m × 8 m (64 m²) | Int. 4 / CBR "A Arena" | Área de busca, define a altitude e o número de passadas |
| Nº de bases de pouso na Fase 1 | 2 | 6 | Int. 6.1 / CBR Fase 1 "A Tarefa" | Mais bases no mesmo quadro e mais buscas por tentativa na CBR |
| Posição das bases | 1 informada, 1 desconhecida | Todas desconhecidas | Int. 6.3 / CBR Fase 1 "Condições" | Busca visual obrigatória nos dois casos |
| Base de pouso | 1 m × 1 m, círculo 0,8 m de diâmetro, cruz 0,5 m | Igual, linhas de 5 cm | Int. 4 Fig. 5 / CBR Fig. 4 | Tamanho conhecido permite estimar distância |
| Base de decolagem | 2 m × 1,5 m, amarelo 1,5 m × 1 m, ~0,5 m de altura | Igual | Int. 4 Fig. 5 e Obs. 3 / CBR Fig. 4 e Obs. 3 | Segunda classe e alvo do pouso final |
| Altura das bases | 0 a 1,5 m | 0 a 1,5 m | Int. 4 Obs. 2, 6.3 / CBR Obs. 2 | Escala aparente varia, altitude não basta |
| Distância mínima base–parede | 0,5 m | 0,5 m | Int. 4 Obs. 3 / CBR Obs. 3 | Parede e rede aparecem junto da base |
| Ambiente da Fase 4 | 2 m × 6 m × 1,5 m, e a base pode ficar no topo | Igual | Int. 4 / CBR "A Arena" | Fundo diferente do piso sob a base |
| Piso | Liso, homogêneo, pode refletir. MDF de qualquer cor | Igual | Int. 4 Obs. 1 / CBR Obs. 1 | Sem textura para odometria visual, com reflexos |
| Iluminação | Luz solar variável, não uniforme | Igual | Int. 4 Obs. 4 / CBR Obs. 4 | Robustez de cor e exposição |
| Tempo | 30 min, até 3 tentativas de até 10 min | Igual | Int. 6.3 / CBR Fase 1 "Condições" | Orçamento de tempo de busca por base |
| Pontos por base | +20 (1ª visita), −5 (visita repetida) | Igual | Int. 6.4 / CBR "Pontuação" | Não gerar detecção duplicada da mesma base |
| Retorno autônomo com pouso | Dobra (2×) | Dobra (2×) | Int. 6.4 / CBR "Pontuação" | Detectar e centralizar na base de decolagem |
| Open-hardware | Dobra (2×) | Dobra (2×) | Int. 6.4 / CBR "Pontuação" | Favorece o X500, Tello não conta |
| Pontuação máxima | 160 (calculado: 2 × 20 × 2 × 2, o edital não informa) | 480 | Int. 6.4 / CBR "Pontuação" | - |
| Drone nas Fases 1 e 2 | ≤ 4 kg sem bateria, ≤ 500 mm entre eixos | Igual | Int. 2 / CBR "O Robô Voador" | Limite de peso para câmera e computador |
| Hover | Altura mínima de 1 m | Igual | Int. 2 / CBR "O Robô Voador" | - |
| **Obrigação:** pousar = todo o trem de pouso na base, hélices desligadas | Sim | Sim | Int. 6.4 / CBR "Pontuação" | Define a precisão do centro |
| **Obrigação:** visitar = identificar **por visão** e pousar | Sim | Sim | Int. 6.4 / CBR "Pontuação" | A detecção precisa ser demonstrável (log/imagem) |
| **Obrigação:** logs/rosbags | Recomendado | Obrigatório se o juiz pedir | Int. 5 Obs. 3 / CBR "O Desafio" Obs. 4 | Registrar as saídas da visão |
| **Proibição:** GPS, RTK, beacons externos | Sim | Sim | Int. 2 / CBR "O Robô Voador" | Visão é a referência de posição |
| **Proibição:** fios e cabos | Sim | Sim | Int. 2 / CBR "O Robô Voador" | Processamento a bordo ou por rádio |
| **Proibição:** alterar código/parâmetros após a 1ª tentativa | Sim | Sim | Int. 5 Obs. 1–2 / CBR "O Desafio" Obs. 1–2 | Nada de recalibrar HSV entre tentativas |
| **Proibição:** medir ou alterar a arena | Sim | Sim | Int. 6.3 Obs., 4 Obs. 5 / CBR "Condições", Obs. 5 | Calibração tem que vir pronta |
| **Proibição:** pousar fora de base (encerra a tentativa) | Sim | Sim | Int. 6.4 Obs. / CBR "Pontuação" Obs. | Falso positivo custa a tentativa |

#### 2.5 Impacto da Plataforma (Teseu/X500 × Tello)

| Aspecto | Teseu (X500) | Tello |
| :--- | :--- | :--- |
| Câmera | Inferior RGB, escolhida e calibrada pela equipe **(ESPERANDO RETORNO DAS SPECS)** | Principal é frontal (720p). Enxergar o chão exige espelho ou a câmera inferior de baixa resolução `[confirmar acesso pelo SDK]` |
| Processamento | A bordo (Raspberry Pi **(ESPERANDO RETORNO DAS SPECS)**): CPU limitada, daí YOLO11n em 320 × 320 e mínimo de 10 FPS | No notebook, com vídeo por Wi-Fi: mais poder de cálculo, mas latência e quedas de vídeo |
| Pontuação | Open-hardware: 2× | Não é open-hardware: sem o 2× |
| Pouso | Trem de pouso largo em base de 1 m: margem de centro pequena | Drone pequeno: margem grande |
| Vibração | Maior, com risco de borrão e rolling shutter | Menor |
| Segurança | Kill switch obrigatório (> 200 g) | Abaixo de 200 g |

O que muda nos requisitos por causa do X500: a meta de FPS/latência (REQ-CV-05) vira o gargalo, a tolerância de centro (REQ-CV-04) fica apertada e é preciso validar com vibração real (REQ-CV-11). Com o Tello, o gargalo passaria a ser a latência do link e o ângulo da câmera.

---

### 3. REQUISITOS DE VISÃO COMPUTACIONAL

> **Prioridade:** **A** (alta, equivale ao P0 do template) | **M** (média) | **B** (baixa). M e B equivalem ao P1.
> "Distância" é sempre câmera–base. As métricas são as do **Interno**, e a CBR só é citada onde a regra dela é diferente.

| ID | Requisito | Métrica (Interno / CBR) | Como verificar | Fonte (seção do edital) | Pri. |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REQ-CV-01** | Detectar a base de pouso (1 m × 1 m) pela câmera inferior | Recall ≥ 80% entre 1,5 m e 3 m de distância | Conjunto de teste anotado | Int. 6.1, 6.2 / CBR Fase 1 "A Tarefa" | A |
| **REQ-CV-02** | Detectar a base no chão e elevada | Recall ≥ 70% com a base a 0 m e a 1,5 m do solo | Conjunto de teste separado por altura da base | Int. 4 Obs. 2, 6.3 / CBR "A Arena" Obs. 2 | A |
| **REQ-CV-03** | Distinguir base de pouso de base de decolagem | 0 confusões decolagem → pouso no conjunto de teste | Matriz de confusão | Int. 6.3 (Interação com o Humano), 4 Fig. 5 / CBR Fig. 4 | A |
| **REQ-CV-04** | Estimar o centro da base de pouso (`dx`, `dy`) | Erro ≤ 10 cm a até 1,5 m de distância (rever com a largura do trem de pouso) | Bancada com deslocamentos medidos (gabarito) | Int. 6.4 (definição de pousar) / CBR "Pontuação" | A |
| **REQ-CV-05** | Rodar o pipeline completo no computador de bordo | Mínimo de 10 FPS (ideal: mais) e latência ≤ 100 ms | Benchmark de 5 min no hardware embarcado | Int. 1 (processamento embarcado), 2 / CBR "Introdução" | A |
| **REQ-CV-06** | Manter a detecção sob iluminação variável sem reajuste | Recall ≥ 70% em cada uma de 2 condições (luz interna e luz de janela), mesmos parâmetros | Conjunto de teste separado por condição de luz | Int. 4 Obs. 1 e 4, 5 Obs. 1 / CBR "A Arena" Obs. 1 e 4 | A |
| **REQ-CV-07** | Não detectar base onde não há base | 0 ativações de `detection_status` em ≥ 2 min de vídeo sem base | Vídeos da arena sem base no quadro | Int. 6.4 Obs. / CBR "Pontuação" Obs. | A |
| **REQ-CV-08** | Manter o centro com a base parcialmente visível na descida | Rastreio até 0,8 m de distância, sem perda > 1 s | Rosbag de descidas até o toque | Int. 6.4 / CBR "Pontuação" | M |
| **REQ-CV-09** | Detectar e centralizar na base de decolagem (2 m × 1,5 m) para o pouso final | Recall ≥ 70% entre 1,5 m e 3 m e erro de centro ≤ 20 cm a até 1,5 m | Conjunto de teste + bancada | Int. 6.4 (retorno autônomo dobra) / CBR "Pontuação" | M |
| **REQ-CV-10** | Funcionar com código e parâmetros congelados | 3 tentativas seguidas em 30 min com o mesmo arquivo de configuração | Ensaio de fase completa cronometrado | Int. 5 Obs. 1–2 / CBR "O Desafio" Obs. 1–2 | A |
| **REQ-CV-11** | Detectar com o drone em movimento de busca | Recall ≥ 70% a 0,3 m/s de velocidade horizontal | Rosbag de voo de varredura | Int. 6.3 (tentativa de 10 min para 2 bases) | M |
| **REQ-CV-12** | Estimar a distância até a base pelo tamanho conhecido do alvo | Erro ≤ 20% entre 0,8 m e 3 m | Bancada com trena, base no chão e elevada | Int. 4 Obs. 2 e Fig. 5 / CBR Obs. 2 e Fig. 4 | M |
| **REQ-CV-13** | Confirmar o alvo antes de sinalizar detecção | Confirmação em 5 de 8 frames, em ≤ 1 s | Teste com vídeo gravado e marcação de tempo | Int. 6.4 Obs. / CBR "Pontuação" Obs. | M |
| **REQ-CV-14** | Registrar as saídas da visão para análise e prova da identificação por visão | 100% das mensagens de `/vision/*` gravadas em rosbag (Interno: recomendado / CBR: exigível pelo juiz) | Inspeção do rosbag após voo | Int. 5 Obs. 3, 6.4 (definição de visitar) / CBR "O Desafio" Obs. 4 | M |

---

### 4. PIPELINE DO DATASET E MODELO DE VISÃO

#### 4.1 Estratégia de Dados e Coleta

* **Classes de Detecção:** `base_pouso`, `base_decolagem`
* **Tamanho e Origem do Dataset:** cerca de 300 imagens anotadas(estimativa, o tamanho real ainda não foi definido).
  * **O que o dataset precisa cobrir:** distâncias diferentes, base no chão e elevada, posições variadas no quadro, mais de uma condição de luz e algumas imagens sem nenhuma base.
  * **Teste:** separar uma parte das imagens só para teste, de sessões de gravação diferentes das de treino, para não avaliar em frames quase iguais aos vistos no treino.
* **Técnicas de Data Augmentation:**
  * Brilho, contraste e HSV.
  * Sombras e manchas de brilho sintéticas.
  * Rotação em qualquer ângulo e espelhamento.
  * Escala e cortes aleatórios (base parcial).
  * Perspectiva leve (inclinação do drone).
  * Desfoque de movimento e ruído.

#### 4.2 Modelo / Algoritmo Escolhido

* **Arquitetura Visual:** **YOLO11n** (320 × 320, exportado para NCNN, mesma combinação YOLO nano + OpenCV da missão passada) para detecção e classificação, seguido de **OpenCV (HSV + contornos + ajuste de elipse)** dentro da caixa detectada para refinar centro e orientação.
  * *Por que híbrido:* a rede lida melhor com luz e fundo variáveis e resolve pouso × decolagem. O refino clássico dá o centro com mais precisão que a caixa do YOLO.
* **Métricas Esperadas (mínimo aceitável):**
  * mAP@50 ≥ 0,80.
  * Recall ≥ 80% entre 1,5 m e 3 m.
  * 0 confusões decolagem → pouso.
  * Erro de centro ≤ 10 cm a até 1,5 m.
  * Mínimo de 10 FPS a bordo.

---

### 5. INTERFACES DE SOFTWARE E TÓPICOS ROS 2

* **Nó ROS 2:** `base_detector_node`
* **Tópicos de Entrada (Inscrição):**
  * `/camera/image_raw` `[sensor_msgs/msg/Image]`: Imagem bruta da câmera inferior.
  * `/camera/camera_info` `[sensor_msgs/msg/CameraInfo]`: Intrínsecos e distorção.
* **Tópicos de Saída (Publicação):**
  * `/vision/target_pose_error` `[geometry_msgs/msg/Vector3]`: `x = dx` (m), `y = dy` (m), `z = dyaw` (rad). O `dyaw` é opcional e não faz parte dos requisitos mínimos.
  * `/vision/detection_status` `[std_msgs/msg/Bool]`: `true` com base confirmada temporalmente.
  * `/vision/detected_command` `[std_msgs/msg/String]`: Classe em foco: `base_pouso` ou `base_decolagem`.
  * `/vision/target_distance` `[std_msgs/msg/Float32]`: Distância estimada até a base, em metros.
  * `/vision/debug_image` `[sensor_msgs/msg/Image]`: Frame com caixa, centro e cruz desenhados.

---

### 6. PLANO DE VERIFICAÇÃO E VALIDAÇÃO (V&V)

| ID Teste | Requisito Relacionado | Procedimento do Teste | Critério de Sucesso | Status |
| :--- | :--- | :--- | :--- | :--- |
| **TST-CV-01** | REQ-CV-01, 02 | Rodar a inferência no conjunto de teste, com a base a 0 m e a 1,5 m do solo e em rotações diferentes. | Recall $\ge 80\%$ entre 1,5 m e 3 m e $\ge 70\%$ em cada altura. | [ ] Não iniciado |
| **TST-CV-02** | REQ-CV-03 | Avaliar a matriz de confusão no conjunto de teste, incluindo frames com as duas classes. | 0 `base_decolagem` classificada como `base_pouso`. | [ ] Não iniciado |
| **TST-CV-03** | REQ-CV-04, 09 | Posicionar a câmera em deslocamentos medidos (0, 10, 20, 40 cm) a 1,0 m e 1,5 m de distância. Comparar com `dx`, `dy`. | Erro $\le 10\text{ cm}$ (pouso) e $\le 20\text{ cm}$ (decolagem). | [ ] Não iniciado |
| **TST-CV-04** | REQ-CV-05 | Medir FPS, latência e uso de CPU com o pipeline completo a bordo, por 5 min contínuos. | FPS $\ge 10$ e latência $\le 100\text{ ms}$. | [ ] Não iniciado |
| **TST-CV-05** | REQ-CV-06 | Repetir TST-CV-01 com luz interna e com luz de janela, sem alterar parâmetros. | Recall $\ge 70\%$ em cada condição. | [ ] Não iniciado |
| **TST-CV-06** | REQ-CV-07, 13 | Rodar o pipeline em $\ge 2$ min de vídeo da arena sem base. Depois, em vídeo com base entrando no quadro. | 0 ativações sem base e confirmação em $\le 1\text{ s}$ com base. | [ ] Não iniciado |
| **TST-CV-07** | REQ-CV-08 | Gravar rosbag de 5 descidas até o toque sobre a base. | Sem perda $> 1\text{ s}$ até 0,8 m de distância. | [ ] Não iniciado |
| **TST-CV-08** | REQ-CV-11 | Voo de varredura a 0,3 m/s sobre a base, gravado em rosbag. | Recall $\ge 70\%$ nos frames com a base no quadro. | [ ] Não iniciado |
| **TST-CV-09** | REQ-CV-12 | Comparar a distância estimada com trena (0,8 m, 1,5 m e 3 m), com a base no chão e elevada. | Erro de distância $\le 20\%$. | [ ] Não iniciado |
| **TST-CV-10** | REQ-CV-10, 14 | Ensaio de fase completa: 30 min, 3 tentativas, mesmo arquivo de configuração, rosbag ligado. | 3 tentativas sem editar nada e rosbag com todos os tópicos `/vision/*`. | [ ] Não iniciado |

---

### 7. DÚVIDAS E RISCOS

#### 7.1 Dúvidas que o edital não responde

1. **Suporte das bases elevadas:** formato e cor do pedestal, e se ele aparece na imagem ao redor da base.
2. **Distância mínima entre as 2 bases:** o edital não informa. Elas podem ficar próximas a ponto de aparecerem juntas no quadro?
3. **Largura das linhas amarelas no Interno:** os 5 cm só aparecem na CBR.

#### 7.2 Riscos

| Risco | Efeito | Mitigação proposta |
| :--- | :--- | :--- |
| Luz do sol e reflexo no piso mudam as cores entre o teste e a tentativa | Perda de detecção com parâmetros congelados | Rede neural na detecção, HSV largo só no refino, dataset em vários horários |
| YOLO não atinge 10 FPS no Raspberry Pi | Controle de pouso instável | Entrada 320 × 320, NCNN |
| Base parcialmente fora do quadro na descida | Perde o centro no momento crítico | Refino por círculo e cruz, e cortes parciais no dataset |
| Base elevada tratada como se estivesse no chão | Erro de escala em `dx`, `dy` | Distância pelo tamanho do alvo (REQ-CV-12) |
| Falso positivo (objeto azul/amarelo, base de decolagem) | Pouso fora de base encerra a tentativa | Classe separada para decolagem, validação geométrica, confirmação temporal |
| Mesma base detectada duas vezes | −5 pontos por visita repetida | Detecções estáveis + controle de bases visitadas na navegação |
| Vibração e movimento borram a imagem | Queda de recall em voo | Fixação com amortecimento, exposição curta, augmentation de blur |
| Piso sem textura degrada a odometria | Posição errada da base "conhecida" | Não confiar só na posição informada, sempre confirmar por visão |