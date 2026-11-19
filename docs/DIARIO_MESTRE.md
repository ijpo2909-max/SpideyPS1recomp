# SPIDEYPS1RECOMP — DIÁRIO MESTRE DE INVESTIGAÇÃO

> **Documento de continuidade técnica.** Fonte organizada para retomar o projeto sem repetir análises, confundir hipóteses com fatos ou misturar resultados de builds/cenas diferentes.

| Metadado | Valor |
|---|---|
| Projeto | `~/Meus Jogos/SpideyPS1recomp` |
| Plataforma de desenvolvimento | Ubuntu 26.04.1 LTS, x86_64 |
| Jogo-alvo | Spider-Man (PS1); comparações pontuais com Spider-Man: Enter Electro |
| Objetivo atual | Controle orbital independente da câmera pelo analógico direito (R3) |
| Última consolidação | 2026-11-19 |
| Estado global | **Investigação técnica em andamento; implementação orbital definitiva ainda não autorizada/validada** |

## Como ler este documento

### Estados de evidência

- **[COMPROVADO]**: observado diretamente em código, disassembly, log, execução ou documentação primária identificada.
- **[PARCIAL]**: há evidência concreta, mas falta estabelecer a interpretação completa ou a cadeia de origem.
- **[HIPÓTESE]**: explicação plausível ainda não demonstrada.
- **[PENDENTE]**: pergunta que ainda precisa ser respondida.
- **[NEGATIVO]**: busca/teste não encontrou o que se procurava; isso não prova inexistência fora do escopo pesquisado.

### Regras para futuras atualizações

1. Preservar datas, caminho dos arquivos, endereços, valores e condições de teste.
2. Nunca converter uma hipótese em fato sem nova evidência.
3. Cada captura de log deve ser identificada por build, cena, sessão, controles usados e intervalo de frames, quando disponível.
4. Não combinar coordenadas de execuções/cenas diferentes em um mesmo cálculo.
5. Antes de pedir um comando, consultar as seções cronológicas e de pendências.
6. Não editar, recompilar ou ativar experimentos que mudem o jogo sem autorização explícita.
7. Trabalhar com comandos pequenos e saída limitada; não usar `rg`.
8. Registrar também resultados negativos, limitações da instrumentação e investigações que não devem ser repetidas.
9. Quando a informação veio de conversa resumida e não de artefato atualmente disponível, indicar essa condição. Este diário não finge possuir logs, prompts do Codex ou relatórios integrais que não foram recuperados.

---

# PARTE I — OBJETIVO E REQUISITOS FUNCIONAIS

## 1. Objetivo de câmera

O objetivo definido pelo usuário é implementar uma câmera orbital controlada pelo R3, mantendo o sistema original do jogo.

Requisitos:

1. **L1 + L3 permanece integralmente original**, incluindo Web Target Mode.
2. **R3 funciona independentemente de L1**.
3. R3 horizontal controla azimute.
4. R3 vertical controla elevação.
5. A distância entre câmera e personagem permanece aproximadamente constante.
6. A câmera continua mirando/enquadrando o Spider-Man.
7. A posição orbital persiste quando o R3 volta à zona neutra.
8. Reaproveitar convenções matemáticas do jogo quando isso for seguro, mas manter o novo controlador separado.
9. Não substituir rotinas MIPS originais nem remapear botões sem evidência e autorização.

## 2. Critérios de sucesso

A implementação só poderá ser considerada validada quando:

- R3 horizontal orbitar continuamente em ambos os sentidos.
- R3 vertical elevar/abaixar a câmera sem perda do alvo.
- Soltar o R3 mantiver o novo ângulo.
- Movimentar o Spider-Man com L3 mantiver o enquadramento.
- L1 + L3 continuar funcionando como no jogo original.
- R3 funcionar sem pressionar L1.
- Não houver interferência indevida em sombra/renderização ou outras matrizes compartilhadas.
- FPS e estabilidade forem medidos e comparados com baseline da mesma cena/build.

---

# PARTE II — LINHA DO TEMPO CONSOLIDADA

> Datas em ordem cronológica. Quando a hora ou o detalhe exato não foi recuperado, isso é explicitado. A cronologia abaixo reúne fatos recuperados de conversas/resumos anteriores e evidências ainda presentes no contexto; não representa uma exportação integral de todas as conversas.

## 2026-10-07 — Pesquisa inicial de projetos e recompilações

### Pesquisa externa: projetos Spider-Man PS1/Linux

- **[COMPROVADO — resultado de pesquisa registrado]** `krystalgamer/spidey-decomp` foi identificado como projeto de descompilação do port de PC, não como um executável Linux jogável.
- **[COMPROVADO — resultado de pesquisa registrado]** PSXRecomp tinha builds Linux para os dois jogos PS1, mas gameplay Linux não estava validado na pesquisa então realizada.
- **[COMPROVADO — resultado de pesquisa registrado]** OpenSpideyPS1 apresentava gameplay, menus, renderer e mods, com releases Windows x64.
- Conclusão da época: não confundir código recompilado/compartilhado com uma versão Linux pronta e jogável.

## 2026-10-11 — Ideia de compartilhar Spider-Man e Enter Electro

- Projeto maior: investigar recompilações nativas de Spider-Man PS1 e Enter Electro e a possibilidade de uma experiência unificada em Linux.
- **[CONCLUSÃO]** Compartilhar runtime/código não torna automaticamente as campanhas integradas.
- Uma coletânea com lançador único foi considerada mais viável do que uma campanha contínua.
- Campanha contínua exigiria integração de saves, menus, carregamento, transições e diferenças entre estruturas dos jogos.
- Essa linha é contexto histórico; o foco atual passou a ser a câmera de Spider-Man PS1.

## 2026-11-13 — Análise de callbacks e entrada de câmera

### 1. Callbacks Spider-Man / Enter Electro

Um script somente de leitura desassemblou:

| Jogo | Callback | Tamanho observado |
|---|---:|---:|
| Spider-Man | `0x80071378` | 2384 bytes |
| Enter Electro | `0x8007D4C4` | 2416 bytes |

- **[COMPROVADO]** Ambos publicam posição em `S1+0x04`, `S1+0x08`, `S1+0x0C`.
- **[COMPROVADO]** Rotinas centrais correspondentes: Spider-Man `0x800701F0`; Enter Electro `0x8007C2D0`.
- **[COMPROVADO]** Conjuntos auxiliares diferem:
  - Spider-Man: `+0x104`, `+0x144`, `+0x214`.
  - Enter Electro: `+0xEC`, `+0x12C`, `+0x1FC`.
- **[PENDENTE]** Identidade semântica exata de todos os vetores/orientações desses campos.

### 2. Codex — sessões identificadas

Foram encontradas duas sessões do Codex, iniciadas aproximadamente às **10:33:29** e **10:54:52** (horário local/conversa de 13/11).

- **[COMPROVADO]** O Codex adicionou `spiderman/patches/CameraInputTrace.cs`, cobrindo SM1/SM2.
- **[COMPROVADO]** Hooks colocados após `func_800701F0` (Spider-Man) e `func_8007C2D0` (Enter Electro).
- **[COMPROVADO]** Ativação por variável `SPIDEY_TRACE_CAMERA`; limite de 4096 registros.
- **[COMPROVADO]** No SM1, a instrumentação lê analógicos em `0x800A5130`, offsets `+4..+7`.
- **[COMPROVADO]** No SM2, usa `0x800B2D04` por meio de `func_80076730`.
- **[PENDENTE]** A resposta real da câmera ainda não havia sido testada quando o resumo foi feito.
- **[COMPROVADO]** Repositório estava em `master`, com commit único `6d7a0c3` de 10/11 (“Mark project as no longer updated”); alterações do Codex estavam no working tree, não commitadas.
- **[COMPROVADO]** A última sessão Codex atingiu limite de uso em `2026-11-13T19:07:43Z`; tentativa-5 foi preparada, mas **não foi lançada**.
- **Limite documental:** os prompts completos e todos os passos das duas sessões Codex não foram recuperados nesta consolidação. Não inventar o conteúdo deles. A existência das sessões e as alterações acima foram recuperadas de conversas anteriores.

### 3. Oito funções auxiliares — Spider-Man e Enter Electro

`main.json` confirmou endereço e tamanho das oito rotinas:

| Papel inferido/confirmado pelo disassembly | Spider-Man | Bytes | Enter Electro | Bytes |
|---|---:|---:|---:|---:|
| Escala vetor 3D | `0x80065D94` | 104 | `0x80070D30` | 104 |
| Soma vetores | `0x80065AC0` | 60 | `0x80070A18` | 60 |
| Consulta tabela trigonométrica e grava vetor | `0x8006C5E4` | 52 | `0x80077B48` | 52 |
| Transformação de vetor 4D por matriz | `0x8006C3CC` | 484 | `0x80077930` | 484 |

- **[COMPROVADO]** O disassembly das oito rotinas foi realizado; Spider-Man e Enter Electro são logicamente muito semelhantes/essencialmente equivalentes nesses auxiliares.
- `0x8006C5E4` / `0x80077B48`: índice `A1`, máscara `0x3FFC`, consulta de tabela trigonométrica e gravação de vetor.
- Tabelas identificadas:
  - Spider-Man: `0x800ACC38`.
  - Enter Electro: `0x800CA6F8`.
- `0x8006C3CC` / `0x80077930`: transforma vetor 4D por matriz fixed-point; produtos e deslocamentos aritméticos `>>12`.
- **[PENDENTE]** Localizar as chamadas e provar a origem dos argumentos no caminho específico da câmera.
- Ferramentas locais identificadas:
  - `/usr/bin/objdump`.
  - `spiderman/tools/mipsdis.py`.
  - `spiderman2/tools/mipsdis.py`.
  - Executáveis extraídos `spiderman/extracted/SLUS_008.75` (732 KB) e `SLUS_013.78` (768 KB).
- Não havia `.sym`/`.map` identificados.
- `main.json` confirma endereços/tamanhos, não substitui o disassembly das instruções.

### 4. Seis endereços não localizados nos mapas principais

- **[NEGATIVO]** Seis endereços SM1 procurados não apareceram nos mapas principais.
- Hipótese levantada: podem pertencer a overlays.
- **[PENDENTE]** Identidade desses endereços não comprovada; não os tratar como inexistentes.

## 2026-11-14 — Continuidade de análise

- Não há detalhes técnicos adicionais suficientemente recuperados nesta consolidação para descrever com segurança o trabalho de 14/11.
- **Regra:** não preencher lacunas com suposições. Se os logs ou a conversa integral forem recuperados, acrescentar o registro datado aqui.

## 2026-11-15 — Origem de ângulo, baseline e escritores da matriz

### 1. Ângulo/heading de câmera e jogador

- JSON `chase-camera-heading-source.json` esperado na investigação estava ausente.
- Documentação/resultado anterior registrava `0x800B5834 + 0x23A` como ângulo-base relativo.
- Referências de snapshots recuperadas:
  - ângulo-base: `1359`.
  - yaw: `2553`.
  - câmera: `2552`.
  - held-right: câmera `2994`.
  - short-right: câmera `2631`.
- **[PENDENTE]** Relação exata entre essas leituras e o heading efetivo da câmera ainda precisa ser revalidada nos arquivos de origem; não assumir que os valores são todos do mesmo frame/snapshot sem conferir o registro original.
- `func_80053898` chama `func_80053A54`; esta usa `ratan2` e normalização `&0x0FFF`.
- `0x8004AD68..0x8004ADD4` calcula entrada com `ratan2` e grava `player+0xF52`.
- **[COMPROVADO]** Esse trecho grava o campo do jogador `player+0xF52`; não é, por si só, uma rotação da câmera.
- **[PENDENTE]** Faltava confirmar a leitura da câmera na continuação de `ptr_8003EC34`.
- Codex estava sem créditos na sessão de 15/11; essa investigação foi relatada como sem alterações de arquivos.

### 2. Baseline geométrico de câmera

Arquivo/log identificado: `spidey-camera-gameplay-baseline-4096.log`.

Valores reportados:
- Spider-Man: `(-19.202, 3.831, -38.073)`.
- Câmera: `(-19.295, 3.643, -38.589)`.
- Escala: 4096 por unidade (Q12 para os valores usados nesse cálculo).
- Separação XZ: aproximadamente `524.2`.
- Separação Y: `188`.
- Distância 3D: aproximadamente `556.89`.
- Delta câmera menos jogador: `(-93.025, -188, -515.88)` (valores arredondados conforme registro).
- Azimute estimado: `-100.22°`.
- Elevação estimada: `-19.73°`.
- **[COMPROVADO COMO CÁLCULO SOBRE O SNAPSHOT]** distância e ângulos foram calculados a partir daquele conjunto de coordenadas.
- **[PENDENTE]** O ponto de mira exato ainda não foi comprovado; convenções de eixo/sinal precisam ser verificadas.
- Fórmula orbital candidata:
  - `Xc = Xp + R cos(θ) cos(φ)`
  - `Yc = Yp + R sin(θ)`
  - `Zc = Zp + R cos(θ) sin(φ)`
- Essa é uma proposta matemática, não a fórmula original demonstrada do jogo.

### 3. Captura de 512 callbacks — origem das escritas da matriz

Arquivo: `.runtime-validation/camera-origin-20261115-165422/console.log`.

- **[COMPROVADO]** Captura contém 512 callbacks completos.
- Cada callback registrou três instruções de escrita MIPS: `0x800761D8`, `0x80076204`, `0x80076214`.
- Essas três instruções produzem nove componentes na faixa `0x800A57CC–0x800A57DD`.
- Em `CameraGteDiagnostics.cs`:
  - `MatrixStart = 0x000A57CC`.
  - `MatrixEndExclusive = 0x000A57DE`.
  - `MaxWritesPerWindow = 128`.
  - `MaxWindows = 512`.
- `updater-window-end writes:0` era contador separado das 4608 `instruction-write` (512 × 9).
- **Importante:** `instruction-write` identifica a instrução MIPS e o endereço escrito. `updater-matrix-write` é outro mecanismo/janela de observação. Não interpretar `updater-window-end writes:0` como ausência de escritas MIPS.
- Ferramentas de análise disponíveis: `spiderman/tools/mipsdis.py` e `xref.py`.
- Não havia símbolos `.sym`/`.map`.
- **[PENDENTE naquela etapa]** usar xref para os endereços e localizar a função que contém as instruções. A identificação foi aprofundada em 16/11.

### 4. Infraestrutura de instrumentação

- `CpuContext` não expõe diretamente o PC atual; tem `EPC`.
- `PSMemory.TrackWritten()` centraliza a captura de escritas de RAM.
- `CallRing.Enter(func.Start)` registra entrada de função, mas não basta para identificar a instrução específica.
- `MipsInstruction.Vram` fica acessível em `FunctionEmitter`/`EmitSingle`, inclusive para delay slots.
- Instrumentação de instrução deve cobrir `SB`, `SH`, `SW`, `SWL`, `SWR` e operações GTE.
- Atenção para evitar duplicação de `SWL/SWR`, pois podem passar por `WriteU32`.
- `PSMemory.TrackWritten()` e `MipsInstruction.Vram` são a base do rastreamento de origem por instrução.
- Essa análise levou à instrumentação de `CameraGteDiagnostics`.

## 2026-11-16 — Testes de câmera, comparação externa e identificação do escritor

### 1. Testes A/B/C de L1 + L3

- Teste C: 1.014 amostras.
- **[COMPROVADO]** Combinações diagonais de L3 enquanto L1 estava pressionado alteraram posição e rotação da câmera.
- Variações observadas, dependentes da orientação inicial:
  - cima + direita: `[-152, -489, +490]`.
  - cima + esquerda: `[+173, -629, +450]`.
  - baixo + esquerda: `[-226, +489, -33]`.
  - baixo + direita: `[+329, +608, -94]`.
- **[COMPROVADO]** L1+L3 horizontal e vertical afetam posição/orientação no modo original.
- **[COMPROVADO]** `held=0x0010` corresponde a L1 no layout usado por `CameraInputTrace`.
- **[BUG DIAGNÓSTICO]** `CameraInputTrace` verificava incorretamente `0x0400` para L1; esse bit não era o correto nesse layout.
- `PadTrace` usa outro layout no qual `0x0400` está correto. Não misturar máscaras entre as duas instrumentações.
- O usuário esclareceu que direções/arcos opostos observados eram movimentos intencionais do L3 enquanto L1 estava pressionado, não movimento espontâneo.
- **[PENDENTE]** corrigir/validar diagnóstico de L1 sem confundir o layout de `PadTrace`.

### 2. Comparação com `krystalgamer/spidey-decomp`

Repositório é descompilação do port PC, não código-fonte equivalente comprovado ao PS1. Serve como referência comparativa, não como autoridade sobre o comportamento MIPS PS1.

Arquivos analisados:
- `camera.cpp`
- `camera.h`
- `algebra.cpp`
- `utils.cpp`
- referências em `ps2funcs.cpp` e documentação PsyQ.

Resultados:
- `CCamera::CM_TripodFocus` usa `Utils_CalcAim` e `M3dMaths_RotMatrixYXZ`.
- Há modos como `CAMERAMODE_LOOKAROUND`.
- `CM_Normal` estava incompleta/TODO.
- Relação direta entre o modo lookaround do PC e L1 no PS1 **não comprovada**.
- `algebra.cpp`/`.h` praticamente vazios.
- `Utils_CalcAim()` calcula deltas XYZ, usa `catan`, calcula ângulos horizontais/verticais, normaliza ângulos em faixa `0..4095` e calcula distância horizontal.
- `Utils_TurnTowards()` aproxima ângulos gradualmente, com wrap e aceleração usando `>>8`.
- `Utils_RotateWorldToObject()` constrói matriz YXZ, transpõe e transforma vetor via GTE.
- `M3dMaths_TransposeMatrix1` foi recuperada.
- `MulMatrix0` usa fixed-point Q12 (`>>12`).
- `FixedXForm` foi recuperada.
- `MulMatrix` apresentava comportamento suspeito.
- `RotMatrixYXZ` no código PC encaminha para endereço fixo `0x0046D1E0`; algoritmo C++ não recuperado.
- `TransMatrix` e `VectorNormal` são stubs no repositório PC.
- Documentação PsyQ consultada confirma convenção de `RotMatrixYXZ`:
  - `R = RY(r1) · RX(r0) · RZ(r2)`.
  - Ângulos 0..4095; 4096 = 360°, 2048 = 180°, 1024 = 90°.
  - Matriz em fixed-point Q12.
  - Aplicação efetiva da composição corresponde a Z, depois X, depois Y.
- **Limite:** isso documenta uma API PsyQ, não prova que a rotina PS1 específica use exatamente a mesma composição em todos os contextos.

### 3. Valores de coordenadas de outra captura

Em outro log/cena, frame 9656:
- Posição bruta do jogador: `(-74442064, 10637312, -109930380)`.
- Escala Q20.12; posição em unidades de mundo:
  - `(-18174.33203125, 2597, -26838.9609375)`.
- Câmera: `[-17852, 2415, -26372]`.
- Diferença aproximada: `(+322.33, -182, +466.96)`.
- Distância aproximada: 600 (registro anterior arredondado; não misturar com baseline de distância 556.89 de outra execução).
- **[COMPROVADO COMO SNAPSHOT DE OUTRA EXECUÇÃO]** esses valores são de outro log/cena; não devem ser combinados com o baseline de 15/11.
- `0x800B5268` continha ponteiro para jogador observado como `0x8011AF8C`; posição do jogador em `player+4,+8,+12`.
- **[HIPÓTESE]** a distância de câmera pode variar conforme cena/estado; não fixar `Radius=600` como verdade sem capturas consistentes.

### 4. Capturas de câmera — frames 2555–2612

Log `spidey.log`, uma execução/cena:
- Frame 2555: inicialização, caller `0x8006FB04`, posição `[0,0,0]`, matriz `[6553,0,0,0,4096,0,0,0,4096]`.
- Frame 2556 em diante: gameplay caller `0x80071CA4`.
- `cameraObject` decimal `2148646836` = `0x8011BFB4`.
- Frame 2556: posição `[-24515,-1342,-6962]`; matriz `[-4096,0,0,0,-4096,0,0,0,4096]`.
- Frames 2558–2612: posição mudou para `[-23802,-1158,-7564]`; matriz também mudou.
- R3 neutro `rx=128`, `ry=128`; L3 deslocado horizontalmente (`lx≈98–101`, `ly=128`); `held=0`.
- `objectPos` ficou `[0,0,0]` e `objectRot` mostrou valores instáveis porque `CameraInputTrace` lia offsets incorretos do contexto `S2=0x8011BFB4`, não do scratch transform.
- Delta de posição da câmera entre frames 2556 e 2612: `(+713,+184,-602)`.
- **[NEGATIVO/CAUTELA]** os campos `objectPos/objectRot` desse trace não são confiáveis para inferir a transformação; os offsets eram incorretos.
- **[CAUTELA]** máscara L1 no `CameraInputTrace` estava errada (`0x0400` em vez de `0x0010`), portanto `l1Held=false` não prova que L1 estava solto.

### 5. Captura de frames 9578–9656

Outra execução/cena, não combinar com frames 2555–2612:
- R3 aproximadamente neutro (`rx=129`, `ry=128`).
- L3 variou em `ly=255` até `128`.
- `held=2048` até frame 9584, depois `0`.
- Posição e matriz global variaram.
- Máscara de L1 do `CameraInputTrace` continuava incorreta, logo o diagnóstico `l1Held` não era confiável.
- Posição do jogador/câmera do frame 9656 está registrada na seção anterior.

### 6. Estado do controlador experimental

Arquivo: `spiderman/patches/CameraOrbitController.cs`.

Parâmetros observados:
- `CameraPosition=0x800A575C`.
- `CameraRotation=0x800A57CC`.
- `Radius=600.0`.
- `Sensitivity=0.025`.
- `PitchLimit=1.15`.
- `Deadzone=12`.
- yaw/pitch acumulados; persistem quando o stick neutraliza.
- `Reset` zera os acumuladores.

Funcionamento experimental descrito:
- Lê matriz como `short / 4096`.
- Usa `-native[6..8]` como direção forward.
- Estima centro orbital a partir de posição da câmera + forward × raio.
- Aplica yaw/pitch, calcula posição/look-at e grava posição global e nove `U16` da matriz.

Problemas identificados:
- Raio 600 não é constante comprovada; outro baseline deu ~556.89.
- Centro estimado pelo forward da matriz não validada, em vez de usar posição do jogador `0x800B5268` e coordenadas Q20.12.
- Orientação/matriz usada não foi validada como a matriz correta para câmera.
- Aloca arrays `double` por frame.
- Executa depois da atualização nativa.
- **Não ativar nem recompilar o experimento sem autorização.**

Arquivos/estado:
- `spiderman/patches/CameraInputTrace.cs` é não rastreado pelo Git (`??`), mas incluído pelo glob `..\patches\**\*.cs` no `SpiderMan.csproj`.
- Backup existente: `CameraInputTrace.cs.bak-20261116-153222`.
- `CameraInputTrace` tem `Enabled = SPIDEY_TRACE_CAMERA=="1"` e `ExperimentEnabled = SPIDEY_CAMERA_EXPERIMENT=="1"`.
- Controller só é chamado quando `ExperimentEnabled`, `LibPad.Analog`, callback gameplay `0x80071CA4` e `pad != 0`.
- A instrumentação GTE sozinha ativa Before/After da trace sem ativar experimento.
- `git diff` vazio não significava arquivo ausente: arquivo não rastreado aparecia em `git status --short --ignored`.

### 7. Investigação de origem de escrita: código MIPS e instrumentação

Arquivo: `tools/RecompOne/RecompOne.Runtime/Diagnostics/CameraGteDiagnostics.cs`.

- `MatrixStart=0x000A57CC`.
- `MatrixEndExclusive=0x000A57DE`.
- `Enabled` por `SPIDEY_TRACE_CAMERA_GTE=1`.
- `MaxWritesPerWindow=128`; `MaxWindows=512`.
- `BeginInstructionWrite(pc,opcode,virtualAddress,gteRegister)` guarda origem por thread.
- `RecordInstructionWrite(effectiveRamAddress,storageSize,ram)` converte endereço virtual (`& 0x1FFFFFFF`), trata escritas parciais SWL/SWR, filtra sobreposição com a região e registra:
  - callback/frame;
  - PC;
  - opcode;
  - endereço virtual;
  - endereço RAM efetivo;
  - largura;
  - registrador GTE (quando aplicável);
  - valor e bytes.
- `PSMemory.TrackWritten()` chama `RecordInstructionWrite` e `WantsWrite`; em seguida, `RecordWrite`.
- A leitura de bytes ocorre depois da escrita MIPS.
- `BeginUpdaterWriteWindow` só abre para callback `0x80071CA4`; janela independente do trace geral.
- Contadores `sequence` de `instruction-write` e `updater-matrix-write` são independentes; correlacionar por frame/endereço/bytes.
- `CameraInputTrace.BeforeUpdate` abre janela quando `CameraGteDiagnostics.Enabled` e caller `0x80071CA4`; `AfterUpdate` fecha a janela antes do experimento orbital.
- Executável já existente: `spiderman/port/bin/Release/net10.0/SpiderMan` (Linux executável) e `SpiderMan.dll`.
- `spiderman/tools/build.sh` apenas invoca `BUILD-LINUX.sh --game 1`; não usar para diagnóstico.
- Variáveis de ambiente não estavam definidas na shell e não havia processo do jogo ativo antes da captura descrita.

### 8. Captura diagnóstica final e identificação do escritor

Execução diagnóstica usada:
```bash
cd "$HOME/Meus Jogos/SpideyPS1recomp/spiderman/port/bin/Release/net10.0" || exit 1
SPIDEY_TRACE_CAMERA_GTE=1 ./SpiderMan
```

Condições solicitadas:
- entrar em fase jogável;
- movimentar personagem normalmente com L3 por alguns segundos;
- R3 centralizado;
- não pressionar L1;
- não controlar manualmente a câmera;
- sair normalmente.

Consulta:
```bash
cd "$HOME/Meus Jogos/SpideyPS1recomp" || exit 1
grep -E '"kind":"(instruction-write|updater-matrix-write|instruction-write-limit|updater-write-limit)"' \
    spiderman/port/bin/Release/net10.0/spidey.log | tail -60
```

Resultado enviado em 2026-11-17:
- Captura mostrou `instruction-write` para:
  - PC `0x800761D8`, opcode `SH`;
  - PC `0x80076204`, opcode `SH`;
  - PC `0x80076214`, opcode `SH`.
- Escritas na faixa `0x800A57CC` até `0x800A57DC`.
- Exemplo frame 1770, callback 434:
  - `0x800761D8` → `0x800A57CC`, valor `0x1984` (bytes `84 19`).
  - `0x80076204` → `0x800A57D2`, valor `0xFF9E` (bytes `9E FF`).
  - `0x80076214` → `0x800A57D8`, valor `0x0144` (bytes `44 01`).
  - Outros stores do mesmo conjunto cobrem os demais halfwords.
- Valores repetiram em frames seguintes.
- Atingiu `instruction-write-limit` com limite 128.
- **[COMPROVADO]** Os três PCs MIPS executam as escritas observadas.
- **[COMPROVADO]** Os stores são `SH` e escrevem três grupos de três halfwords.
- **[PENDENTE]** provar se essa região é a matriz da câmera em todos os usos; há evidência de consumidores GTE, mas a região pode ser compartilhada.
- **[PENDENTE]** rastrear de onde vêm os operandos e qual função/contexto chama esse bloco.

### 9. Trecho de `main.cs` localizado

Arquivo: `spiderman/generated/main.cs`.

Busca:
```bash
grep -n -E '800761D8|80076204|80076214' spiderman/generated/main.cs | head -30
```

Resultado:
- linha 215471: store `SH` PC `0x800761D8`, endereço `c.A1`, valor `c.V0`.
- linha 215481: store `SH` PC `0x80076204`, endereço `c.A3`, valor `c.V1`.
- linha 215485: store `SH` PC `0x80076214`, endereço `c.T1`, valor `c.A0`.

Trecho examinado:
```csharp
L800761C4: ;
RecompOne.Runtime.Hardware.GteScreen.LoadU16(c, 2, m, c.S0, true);
{ var _r = (long)(int)c.V0 * (int)c.T4; c.LO = (uint)_r; c.HI = (uint)(_r >> 32); }
c.V0 = c.LO;
c.SetDerived(2, (uint)((int)c.V0 >> 12), 2);
{ uint _storeAddress = c.A1; ... m.WriteU16(_storeAddress, (ushort)c.V0); }

RecompOne.Runtime.Hardware.GteScreen.LoadU16(c, 3, m, c.A2, true);
{ var _r = (long)(int)c.V1 * (int)c.T3; c.LO = (uint)_r; c.HI = (uint)(_r >> 32); }
...
c.V1 = c.LO;
c.SetDerived(3, (uint)((int)c.V1 >> 12), 3);
{ uint _storeAddress = c.A3; ... m.WriteU16(_storeAddress, (ushort)c.V1); }

RecompOne.Runtime.Hardware.GteScreen.LoadU16(c, 4, m, c.T0, false);
...
{ uint _storeAddress = c.T1; ... m.WriteU16(_storeAddress, (ushort)c.A0); }
...
if (c.V0 != 0u) goto L800761C4;
```

- **[COMPROVADO]** O laço `L800761C4` executa operações de multiplicação fixed-point e grava componentes em três destinos.
- **[COMPROVADO]** Há operações GTE `Execute`/`ReadTo` antes do laço.
- **[PARCIAL]** O padrão é compatível com uma transformação geométrica por matriz.
- **[NÃO COMPROVADO]** Não está demonstrado que o bloco calcula exclusivamente orientação da câmera. Pode ser transformação usada em renderização ou matriz compartilhada.
- O trecho pedido `215420–215510` mostrou código dentro do bloco, mas o cabeçalho da função não foi mostrado. A solicitação seguinte para `215300–215420` foi interrompida pela lembrança do usuário de que já havíamos identificado os registradores em conversas anteriores.

### 10. Recuperação de investigação anterior sobre `func_80075D0C`

Uma conversa anterior recuperou:
- **[COMPROVADO segundo histórico anterior]** o bloco pertence a `func_80075D0C`.
- **[COMPROVADO segundo histórico anterior]** prólogo/cópias de registradores já tinham sido examinados.
- `A0` preservado em `S5`.
- `A1` preservado em `S4`.
- `A2` preservado e também salvo em `GP+0x1150`.
- Chamador registrado em `0x8002BE0C`.
- **[DIVERGÊNCIA A RESOLVER]** outro levantamento indicou vários contextos/call sites de renderização. Não afirmar que `0x8002BE0C` é o único chamador até fazer xref completo e distinguir os caminhos.
- Valores/descrição de parâmetros não devem ser completados por inferência. O próximo passo é recuperar os registros originais ou consultar o disassembly e xref já disponíveis, sem repetir a descoberta do prólogo.

---

# PARTE III — MAPA TÉCNICO CONSOLIDADO

## 3.1 Endereços e estruturas

| Endereço | Papel observado | Observação |
|---|---|---|
| `0x800A575C` | posição global da câmera (X) | Y/Z em `+4`, `+8` |
| `0x800A5734` | destino de transformação em `func_800701F0_Impl` | não é o mesmo que `0x800A57CC` |
| `0x800A57CC–0x800A57DD` | 18 bytes escritos por três `SH`; lidos como matriz por consumidores GTE | não assumir que é exclusivamente a câmera |
| `0x800B5268` | global contendo ponteiro do jogador | jogador observado em `0x8011AF8C` |
| `player+0x04/+0x08/+0x0C` | coordenadas do jogador | Q20.12 em logs de gameplay |
| `0x800B5834+0x23A` | ângulo-base relativo citado em documentação anterior | origem/consumo ainda precisam de confirmação |
| `0x800ACC38` | tabela trigonométrica Spider-Man | usada pela função `0x8006C5E4` |
| `0x800CA6F8` | tabela trigonométrica Enter Electro | usada pela função `0x80077B48` |
| `0x800A5130` | buffer/estrutura de analógicos lida pelo trace SM1 | offsets `+4..+7` |
| `0x800B2D04` | dados de analógicos lidos pelo trace SM2 | via `func_80076730` |
| `0x8011BFB4` | contexto de câmera observado em callback gameplay | `A0`/`S2` no contexto descrito; não confundir com scratch transform |

## 3.2 Funções MIPS e auxiliares

| Função/endereço | Resultado conhecido | Estado |
|---|---|---|
| `func_800701F0` | wrapper com Before/After e chamada à implementação | comprovado |
| `func_800701F0_Impl` | atualiza posição global e transformação | parcialmente analisada |
| `func_8006C64C` | converte quatro componentes em matriz 3×3 | comprovado o fluxo; semântica quaternion não comprovada |
| `func_8008BEE4` (`TransMatrix`) | copia três words da origem para offsets de translação | comprovado |
| `func_80075D0C` | contém o laço que escreve nove halfwords em `0x800A57CC..` | comprovado segundo disassembly/histórico |
| `func_80053898` | chama `func_80053A54` | comprovado no histórico |
| `func_80053A54` | usa `ratan2`, normaliza com `&0x0FFF` | comprovado no histórico |
| `0x8004AD68..0x8004ADD4` | calcula entrada e grava `player+0xF52` | não é giro da câmera |
| `0x80065D94` / `0x80070D30` | escala vetor 3D SM1/SM2 | disassembly comprovado |
| `0x80065AC0` / `0x80070A18` | soma vetores SM1/SM2 | disassembly comprovado |
| `0x8006C5E4` / `0x80077B48` | consulta tabela trigonométrica e grava vetor | disassembly comprovado |
| `0x8006C3CC` / `0x80077930` | transformação de vetor 4D por matriz Q12 | disassembly comprovado |

## 3.3 Registradores já recuperados

### `func_800701F0_Impl`

- `S2=A0` = contexto recebido, observado `0x8011BFB4`.
- `S0=0x800A0000` scratch auxiliar.
- `S1=0x800A5734` destino da transformação.
- origem para `TransMatrix`: `S0+4 = 0x800A0004`.
- vetor origem recebe dados derivados de `S2+4`, `S2+8`, `S2+0xC`, escalados `>>12`.
- `TransMatrix`: `A0=S1=0x800A5734`, `A1=S0+4=0x800A0004`.
- translação da transformação em `S1+0x14`, `+0x18`, `+0x1C` = `0x800A5748`, `0x800A574C`, `0x800A5750`.
- posição global da câmera em `0x800A575C`, `+4`, `+8` deriva de `S2+0x1B8`, `+0x1BC`, `+0x1C0`.
- `func_8006C64C` grava matriz no destino da transformação `S1=0x800A5734`, não diretamente no objeto `S2`.

### `func_80075D0C`

- `A0` preservado em `S5`.
- `A1` preservado em `S4`.
- `A2` preservado e também salvo em `GP+0x1150`.
- Chamador registrado: `0x8002BE0C`.
- **Atenção:** esses dados vieram de investigação anterior recuperada. Confirmar call sites completos e argumentos antes de usá-los como explicação do caminho de câmera.

## 3.4 Convenções numéricas

- Várias rotinas usam fixed-point Q12: `4096` representa 1.0; multiplicação seguida de `>>12`.
- Documentação PsyQ consultada: ângulos de rotação em faixa de 0 a 4095 para uma volta completa.
- `ratan2` e `&0x0FFF` aparecem em `func_80053A54`.
- Coordenadas do jogador em logs podem ser Q20.12 (raw dividido por 4096).
- Não presumir que toda matriz na região global está em Q12 apenas porque parte do jogo usa Q12. Os valores observados em `0x800A57CC` incluem valores acima de 4096 em capturas antigas; validar formato e layout por rotina consumidora.

---

# PARTE IV — EXPERIMENTOS: RESULTADOS E LIMITAÇÕES

## 4.1 Testes anteriores de controle/câmera

| Experimento/build | Resultado registrado | Interpretação |
|---|---|---|
| `CameraStickExperiment` | ~27 FPS; bússola respondeu; câmera não orbitou | experimento não validou órbita |
| `CameraOrbitController` (teste anterior) | personagem saiu do enquadramento; órbita incorreta; em um teste ~5 FPS | controlador experimental incorreto/ineficiente |
| `.runtime-validation/spiderman-camera-manual-20261113/attempt-4` | baseline ~30 FPS | referência de desempenho daquela execução |
| Outra build de teste | ~17 FPS; R3 não moveu câmera; bússola também não se moveu | não generalizar resultado para outras builds |
| Capturas de 16/11 | R3 neutro; L3 moveu personagem; posição/matriz global variaram | câmera nativa acompanha/atualiza durante gameplay; máscara L1 inválida impede concluir estado L1 |

**Regra:** nunca dizer “a bússola responde ao R3” sem indicar qual build/teste. Os resultados foram inconsistentes entre builds.

## 4.2 Problemas conhecidos do trace

- `CameraInputTrace` lia `objectPos/objectRot` em offsets errados do contexto `S2`; esses campos não representam o scratch transform.
- `l1Held`/`l1Pressed` usavam `0x0400`, mas o bit L1 no layout desse trace é `0x0010`.
- `PadTrace` usa layout diferente, no qual `0x0400` é válido.
- Um valor `l1Held=false` produzido pelo trace antigo não prova que L1 estava solto.
- Capturas de frames 2555–2612, 9578–9656 e frame 1770 são execuções distintas; não misturar.
- `instruction-write` e `updater-matrix-write` têm contadores e janelas independentes.

---

# PARTE V — PESQUISA EXTERNA E REFERÊNCIAS

## 5.1 `krystalgamer/spidey-decomp`

- Projeto de descompilação do port PC, não fonte original PS1.
- Útil para recuperar ideias matemáticas e nomenclatura de câmera; não prova equivalência de algoritmos ou layout de memória no PS1.
- `CCamera::CM_TripodFocus`, `Utils_CalcAim`, `M3dMaths_RotMatrixYXZ`, `CAMERAMODE_LOOKAROUND` e utilitários foram analisados.
- `CM_Normal` incompleta.
- `RotMatrixYXZ` encaminha a endereço fixo PC `0x0046D1E0`; corpo não recuperado em C++.
- Não assumir que o modo `LOOKAROUND` é o mesmo comportamento de L1+L3 do PS1.

## 5.2 Documentação PsyQ

- `RotMatrixYXZ`: `R = RY(r1) · RX(r0) · RZ(r2)`.
- Ângulos: 4096 = 360°, 2048 = 180°, 1024 = 90°.
- Matriz fixed-point Q12.
- Referência de API; não prova automaticamente a implementação de qualquer rotina específica do jogo.

## 5.3 Outras pesquisas de projetos

- PSXRecomp: builds Linux para Spider-Man e Enter Electro foram encontrados na pesquisa histórica; gameplay Linux não estava validado naquele momento.
- OpenSpideyPS1: gameplay, menus, renderer e mods; releases Windows x64 encontrados.
- `krystalgamer/spidey-decomp`: referência PC.
- Não há neste diário prova de que qualquer um desses projetos forneça uma implementação orbital PS1 pronta.

---

# PARTE VI — ESTADO DO CODEX E DO GIT

## 6.1 Codex

- Sessões identificadas em 13/11, por volta de 10:33:29 e 10:54:52.
- Adição de `CameraInputTrace.cs` (SM1/SM2), hooks pós-rotina de câmera e variável `SPIDEY_TRACE_CAMERA`.
- Última sessão atingiu limite em `2026-11-13T19:07:43Z`; tentativa-5 preparada, mas não lançada.
- Outras análises em 15/11 ocorreram quando Codex estava sem créditos; foi registrado que aquela investigação não alterou arquivos.
- **Lacuna:** não foram recuperados os prompts completos, saídas integrais nem o histórico completo de alterações do Codex. O diário só registra o que foi confirmado em conversas/resumos anteriores.

## 6.2 Git

- Branch `master`.
- Único commit conhecido: `6d7a0c3` em 2026-11-10, mensagem “Mark project as no longer updated”.
- Alterações feitas pelo Codex estavam no working tree, não commitadas, na inspeção de 13/11.
- `CameraInputTrace.cs` depois apareceu como `??` (não rastreado).
- `git diff` vazio não significa ausência de arquivo: arquivos não rastreados não aparecem no diff normal.
- Não presumir estado atual do Git sem executar `git status`; este registro descreve apenas a inspeção histórica.

---

# PARTE VII — PERGUNTAS EM ABERTO E PRÓXIMA SEQUÊNCIA

## 7.1 Pendências prioritárias

1. **Resolver a cadeia de `func_80075D0C`:** recuperar o prólogo/epílogo e xref já investigados; confirmar todos os call sites e resolver a divergência entre o chamador `0x8002BE0C` e múltiplos contextos de renderização.
2. **Rastrear operandos do laço `0x800761C4`:** de onde vêm os valores lidos via `S0`, `A2`, `T0` e os registradores `T3`, `T4`, `V0`, `V1`, `A0`; identificar se os dados são uma matriz de transformação, projeção ou outra estrutura.
3. **Mapear todos os consumidores de `0x800A57CC`:** separar câmera, renderização, sombra e outros usos. O fato de uma rotina escrever ali não prova propriedade exclusiva da câmera.
4. **Correlacionar em uma mesma execução:** posição do jogador, posição real da câmera, matriz efetivamente carregada na GTE, frame e callback.
5. **Entender `0x800B5834+0x23A` e `ptr_8003EC34`:** confirmar heading e leitura de câmera; recuperar o JSON ausente ou refazer captura controlada.
6. **Corrigir interpretação de L1 no trace** (bit `0x0010` no layout de `CameraInputTrace`) somente com autorização para editar; alternativamente, documentar e interpretar manualmente os dados brutos.
7. **Validar a matriz de orientação e convenção de eixos/sinais** antes de qualquer gravação experimental.
8. **Só então** especificar e implementar controlador orbital R3 separado.
9. Medir FPS e comportamento em build controlada, sempre comparando mesma cena e condições.

## 7.2 Próximo passo mínimo recomendado

Não pedir novamente o prólogo como se nunca tivesse sido visto.

Primeiro, localizar no histórico/artefatos existentes a análise anterior de `func_80075D0C`, incluindo:
- início e fim da função;
- prólogo e registradores preservados;
- xref/lista completa de call sites;
- contexto de chamada `0x8002BE0C`;
- origem dos valores em `S0`, `A2`, `T0`, `T3`, `T4`;
- qualquer anotação do Codex sobre esses registradores.

Se os artefatos originais não estiverem disponíveis, fazer apenas uma consulta de leitura pequena para recuperar a região/call sites específicos, sem repetir a investigação inteira.

---

# PARTE VIII — REGISTROS FUTUROS

Copiar este modelo ao final do documento a cada investigação relevante:

```markdown
## Registro — AAAA-MM-DD — título curto

**Pergunta/objetivo:**  
**Contexto anterior consultado:**  
**Build/cena/sessão:**  
**Comandos e arquivos examinados:**  
**Evidências diretas:**  
**Resultados numéricos/logs:**  
**Conclusão comprovada:**  
**Hipóteses ainda abertas:**  
**Resultados negativos/limitações:**  
**Pendências:**  
**Próximo passo mínimo:**  
**Arquivos alterados:** nenhum / listar explicitamente  
**Autorização do usuário:** diagnóstico / edição / build / teste
```

## Índice rápido de consultas

| Quero saber… | Consultar |
|---|---|
| Objetivo exato da câmera | Parte I |
| O que já foi feito em 13/11 | Linha do tempo — 13/11 |
| Resultados da pesquisa externa/PsyQ | Parte V |
| O que o Codex fez | Parte VI |
| O que foi medido em 15/11 | Linha do tempo — 15/11 |
| Testes de L1+L3, logs e matriz em 16/11 | Linha do tempo — 16/11 |
| Endereços e registradores | Parte III |
| Por que não confiar em alguns campos do trace | Parte IV |
| O que falta fazer | Parte VII |

---

# NOTA DE INTEGRIDADE HISTÓRICA

Este documento é uma consolidação de informações recuperadas das conversas anteriores, resumos de memória, resultados citados e evidências presentes no contexto atual. **Não é uma exportação integral de todos os chats, todos os prompts do Codex ou todos os arquivos/logs históricos.**

Quando um detalhe não foi recuperado com segurança, ele foi marcado como lacuna ou pendência. Para tornar este diário ainda mais completo no futuro, podem ser incorporados os arquivos originais de log, relatórios do Codex, diffs e notas antigas — sempre preservando a origem e a data de cada evidência.

---

# ADENDO — 2026-11-19 — Rastreamento do ponteiro global 0x800B54A8

Este adendo registra a continuação da investigação depois da consolidação das Partes I–VIII. O conteúdo anterior foi preservado; este registro acrescenta evidências obtidas em 19/11.

## 1. Instrumentação e condições

Foram feitas alterações temporárias em:

- `tools/RecompOne/RecompOne.Runtime/Diagnostics/CameraGteDiagnostics.cs`
- `tools/RecompOne/RecompOne.Runtime/Memory/PSMemory.cs`

Backups criados antes das alterações:

- `CameraGteDiagnostics.cs.bak-pointer-20261119-183647`
- `PSMemory.cs.bak-pointer-20261119-183647`

A captura foi ativada com `SPIDEY_TRACE_POINTER=1`. O usuário entrou em gameplay, manteve R3 centralizado, não pressionou L1 intencionalmente e caminhou com L3. O jogo foi encerrado normalmente.

A compilação de diagnóstico foi concluída com sucesso por:

```bash
dotnet build spiderman/port/SpiderMan.csproj -c Release --no-restore
```

Resultado informado: 12 warnings e zero erros.

## 2. Escritor identificado

**[COMPROVADO pela captura de escrita]**

- Endereço virtual monitorado: `0x800B54A8` (RAM física correspondente: `0x000B54A8`).
- Instrução escritora: PC MIPS `0x800612D8`.
- Opcode: `SW`.
- A captura identificou a origem da escrita (`originKnown: true`).
- Os valores observados alternaram entre `0x8009A6E4` e `0x8009A75C`.
- Diferença entre os dois endereços: `0x78` (120 bytes).

## 3. Rotina responsável: func_800612B8

A instrução pertence a `func_800612B8`. O fluxo relevante do código recompilado é equivalente a:

```c
V0 = 0x800A0000;
V1 = *(GP + 0xCB4);
BASE = V0 - 0x591C;       // 0x8009A6E4

if (V1 != BASE)
    V0 = BASE;
else
    V0 = BASE + 0x78;     // 0x8009A75C

A0 = *(V0 + 0x70);
*(GP + 0xCB4) = V0;       // global 0x800B54A8
A1 = 0x1000;
ClearOTagR(A0, A1);
```

Os endereços resultam de:

- `0x800A0000 - 0x591C = 0x8009A6E4`
- `0x8009A6E4 + 0x78 = 0x8009A75C`

A função escolhe uma das duas estruturas com base no ponteiro anteriormente armazenado, lê o campo no deslocamento `+0x70`, grava o endereço da estrutura selecionada no global e chama `ClearOTagR`.

## 4. Interpretação e limites

- **[COMPROVADO]** O global `0x800B54A8` recebe alternadamente os endereços-base `0x8009A6E4` e `0x8009A75C`.
- **[COMPROVADO]** A rotina chama `ClearOTagR` logo depois da seleção da estrutura.
- **[PROVÁVEL, NÃO DEFINITIVO]** O fluxo é compatível com alternância entre estruturas/buffers de renderização associados à Ordering Table do PS1.
- **[NÃO COMPROVADO]** Não há evidência para chamar `0x800B54A8` de ponteiro da câmera. A presença desse global em outro caminho que prepara argumentos para `func_80075D0C` não estabelece propriedade semântica de câmera.
- **[PENDENTE]** Inspecionar os campos das estruturas em `0x8009A6E4` e `0x8009A75C`, especialmente `+0x70`, e acompanhar o valor carregado até seus consumidores.

## 5. Localização de ClearOTagR

No código gerado `spiderman/generated/main.cs` foram localizados:

- chamada `SpiderMan.ClearOTagR(c, m)` por volta da linha 187688;
- wrapper `public static void ClearOTagR(CpuContext c, IMemory m)` por volta da linha 251354;
- o wrapper encaminha para `Recompiled.GpuPatches.ClearOTagR(c, m)`.

Uma busca limitada a `tools/RecompOne` por `ClearOTagR` não retornou resultados. Isso é apenas um resultado negativo daquele escopo; a implementação ainda precisa ser localizada no repositório completo.

## 6. Próximo passo mínimo

1. Localizar a implementação real de `Recompiled.GpuPatches.ClearOTagR`.
2. Examinar os dois blocos de 120 bytes e os ponteiros em `+0x70`.
3. Acompanhar o valor de `mem32[mem32[0x800B54A8] + 0x70]` até a chamada de `func_80075D0C`.
4. Só então determinar se há qualquer relação funcional com a matriz observada em `0x800A57CC`.

**Regra de continuidade:** não repetir a captura de escrita que já identificou PC `0x800612D8`, a menos que uma nova hipótese exija outra condição de teste. Não promover a hipótese de double buffering a fato sem examinar a implementação de `ClearOTagR` e os campos das estruturas.
