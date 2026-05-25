// - Um bom script acerta sinais. Um script científico explica sinais. Um trader lucrativo usa ambos
// === MAO — Somente Sinais (NTSL) — CONSOLIDADO ===
// Fluxo EXTREMAMENTE ORGANIZADO para garantir que tudo funcione como um "organismo vivo" autoajustável.
// Correções e Melhorias sempre baseadas no Manual:
// - VRS Dinâmico: usa limiar manhã/tarde (inputs VRS_Dinamico / VRS_Manha / VRS_Tarde).
// - Filtro de Candle: bloqueia doji via C_Doji(DojiPercent) e aplica bodyATR / wickRatio.
// - ORB: disparo condicionado a horarioPermitido e VRS dinâmico; inclui candleOk.
// - Mantidas as regras do Manual para PaintBar/PlotText e demais funções.
// Funcionamento:
// - Pintar Verde na tela quando para efetuar a Compra, mantendo verde ate aparecer Amarelo que sera para vender e realizar o resultado (lucro)
// - Pintar Vermelho na tela quando para efetuar a Venda, mantendo vermelho ate aparecer Amarelo que sera para comprar e realizar o resultado (lucro)
// - Reset no candle seguinte ao amarelo (1 candle neutro garantido) → saída clara e transição limpa.
// - Elimina duplicidades de blindagem/persistência → uma única persistência determinística por barra, ou seja, pinta somente uma cor por candle, a correta
/////////////////////////////// INPUTS /////////////////////////////////////////
input
  VRS_TP_Manha(2.00);
  VRS_SL_Manha(0.85);
  VRS_TP_Tarde(1.40);
  VRS_SL_Tarde(0.75);
  VRS_Cooldown_Tarde(3);
  // barras extras de cooldown na tarde
  VRS_JanelaMorta(true);
  PG_ATR_Breakeven_Trigger(1.00);
  // quando lucro >= +1.0 ATR
  PG_ATR_Breakeven_Offset(0.15);
  // 0.10–0.20 ATR
  PG_TimeStop_Bars(9);
  // 8–10 barras
  PG_TimeStop_MinATR(0.50);
  // se não andou +0.5 ATR até N barras: breakeven/saída parcial
  WickAlpha(1.20);
  // α para wick-ratio (1.0 a 1.5)
  Periodo(18);
  Desvio(1.8);
  ConfirmacaoTendencia(50);
  PeriodoIFR(12);
  PeriodoInclinacao(5);
  ATR_Periodo(12);
  ADX_Periodo(12);
  ADX_Media(14);
  MinADX(18.0);
  RatioBWATR_Min(0.32);
  SlopeMin(0.00);
  TP1_Mult(3.2);
  TP2_Mult(5.0);
  SL_Mult(0.55);
  MaxBarrasTrade(18);
  CooldownBars(1);
  VRS_Periodo(18);
  MinVRS(0.85);
  VRS_Dinamico(true);
  VRS_Manha(1.10);
  VRS_Tarde(0.65);
  HabilitarAcelerador(true);
  Acelerador_ADX_MinDelta(2.0);


  AcelIgnoraTrend(true);
  UsarIFR_Extremos(true);
  IFR_FalsoRomp_Compra(80);
  IFR_FalsoRomp_Venda(20);
  StopHibrido(true);
  StopModo(0);
  WickRatioMax(1.20);
  MinBody_ATR(0.12);
  BloquearDoji(true);
  DojiPercent(5);
  Confirm2Steps(true);
  ConfirmaCloseBB(true);
  SomenteSinais(true);
  ColorirPorPosicao(false);
  PlotarMarcadores(true);
  PintarOportunidade(true);
  CorOpCompra(clGreen);
  CorOpVenda(clRed);

  CorSaida(clYellow);

  // Produção: desligado (menos latência, menos ruído visual).
  // Ative só para auditoria/depuração.
  ModoDebug(false);
  UsarXRay(false);

  // ================= WALK-FORWARD / FORA DA AMOSTRA / ANTI-OVERFITTING =================
  WF_Ativo(true);
  WF_BarrasTreino(240);
  WF_BarrasOOS(80);

  WF_MinTradesTreino(6);
  WF_MinWinRate_Treino(0.52);
  WF_MinPF_Treino(1.15);
  WF_MinExp_Treino(0.08);
  WF_MaxDD_Treino(2.80);

  WF_MinTradesOOS(4);
  WF_MinPF_OOS(1.00);
  WF_MinExp_OOS(0.00);
  WF_MaxDD_OOS(3.20);

  WF_RequerOOSPositivo(true);
  WF_PenalidadeOOS(8.0);
  WF_BloqueioBarras(12);

  FiltroGapAbertura(true);
  MaxGapPercent(1.5);

  // Disciplina de janela focada em liquidez real da B3
  FiltroHorario(true);
  HoraInicio(900);    // 09:00 no formato HHMM, conforme Manual NTSL
  HoraFim(1800);      // 18:00 no formato HHMM, conforme Manual NTSL
  BloquearPrimeiraHora(false); // mantemos a 1ª hora liberada p/ capturar gaps e momentum da abertura

  RegimeAware(true);
  RiskAdapt(true);
  HabilitarORB(true);
  // ORB Adaptativo por Regime
  ORB_MinClamp(10);
  ORB_MaxClamp(25);
  ORB_K(8.0);
  // ganho por rank de vol (ajustável)
  ORB_RequerClose(true);
  // exige confirmação no FECHAMENTO do candle (reduz falso rompimento intrabar; coerente com "SomenteSinais")

  ORB_PermiteReteste(true);
  ORB_StopReteste_ATR(0.70);
  ORB_RequerVRS(true);
  // VRS OBRIGATÓRIO no ORB: atende ao Manual (VRS dinâmico + candleOk + horário), reduz falsos rompimentos

  HabilitarPDH_PDL(true);
  HabilitarBBMidBounce(true);
  HabilitarNR(true);
  NR_Barras(4);
  NR_Fator(0.6);
  HabilitarGapFade(true);
  GapFade_MinPercent(0.8);
  GapFade_TargetModo(0);
 
  // Quantidade base continua existindo como fallback seguro,
  // porém a execução passa a usar qtyEntryEff (dinâmico).
  // Nunca operar com 0.
  QtdOrdem(1);

  // ================= POSITION SIZING DINÂMICO =================
  // Baseado em regime, volatilidade, saúde do auditor e pressão do WF.
  // Mantém determinismo e reduz alavancagem justamente quando a borda piora.
  SizingDinamico(true);
  QtdMin(1);
  QtdMax(3);
  Sizing_AuditGain(0.30);     // reforço quando o auditor está saudável
  Sizing_AuditPain(0.40);     // redução quando dor / DD crescem
  Sizing_WFPenalty(0.50);     // redução quando o WF degrada
  Sizing_VolAltaRedutor(1);   // corta 1 contrato em vol alta
  Sizing_RegimeTrendBonus(1); // adiciona 1 contrato em tendência limpa

  MaxTradesDia(6);
  StopDia_Loss(900.0);
  StopDia_Gain(1600.0);
  Kill_ConsecLoss(3);
  CooldownLoss_Bars(5);

  Trail_ATR_Fator(0.35);
  BreakEven_ATR(0.25);
  Usar_OCO_ToCover(true);

  // ================= EXECUÇÃO REAL / CUSTOS / CONTRATO DE FILL =================
  // Tudo em TICKS equivalentes para manter a unidade compatível com o PnL do script.
  Exec_UsarCustosReais(true);
  Exec_SlippageIn_Ticks(1.0);
  Exec_SlippageOut_Ticks(1.0);
  Exec_Spread_Ticks(0.5);
  Exec_Delay_Ticks(0.5);
  Exec_Corretagem_Ticks(0.0);
  Exec_Emolumentos_Ticks(0.0);

  // Piso líquido mínimo: o trade só merece agressão se o edge superar custo real.
  Exec_MinNet_ATR(0.12);
  Exec_PenalidadeVolAlta_Ticks(0.5);
  Exec_PenalidadeOOS_Ticks(0.5);

  // ================= CONTRATO CIENTÍFICO DE PRODUÇÃO / OVERFITTING / CAPTURA =================
  // Estes parâmetros não criam novo módulo: apenas tornam mensurável e bloqueável
  // o que antes era subjetivo. O objetivo é impedir lucro "bonito no histórico"
  // e frágil em execução real.
  WF_MaxOverfitGap(0.35);          // diferença máxima aceitável treino - OOS em ATR/trade
  WF_MinStabilityScore(0.55);      // estabilidade mínima combinando PF, expectativa e DD
  WF_MinPromotionScore(0.62);      // nota mínima para promover o fold
  WF_MinFoldsApproved(2);          // folds consecutivos aprovados para produção
  WF_MaxFoldsRejected(2);          // rejeições consecutivas acionam proteção
  WF_PenalidadeOverfit(4.0);       // penalidade extra quando treino > OOS demais

  Exec_MaxPendingBars(2);          // barras máximas aguardando fill/cancelamento
  Exec_MaxFillDrift_Ticks(3.0);    // divergência máxima entre AMARELO visual e execução estimada
  Exec_StopOffsetMin_Ticks(30.0);  // piso de offset do stop-limit em ticks
  Exec_StopOffsetATR_Fator(0.12);  // offset adicional proporcional ao ATR em eventos hostis
  Exec_GapEmergency_ATR(1.20);     // gap extremo em ATR
  Exec_GapEmergency_Ticks(8.0);    // folga extra em gap/evento extremo

  Capture_MinEfficiency(0.55);     // captura mínima do extremo teórico até a saída real
  Capture_MaxGiveBack_ATR(0.75);   // devolução máxima permitida a partir do topo/fundo
  Capture_MinNetATR(0.05);         // lucro líquido mínimo para validar AMARELO como útil

  Prod_BloquearExecucaoReal(true); // sem promoção WF/OOS, não deixa automatizar entrada real
  Prod_MinPromotionScore(0.62);
  Prod_RequerFoldsAprovados(2);
  Prod_BloqueiaSeRejeicoes(2);

  //////////////////////////// VARIÁVEIS /////////////////////////////////////////
var
  // ==== Correções: variáveis globais exigidas pelo Manual (NTSL) ====
  dir                                                 : Integer;
  // +1 comprado | -1 vendido
  pnlATR,moveATR,bePrice,beCushion,targetOffset,trailDist,trailBase,slCandidate,candidateFloor,candidateCeil,minGainExitATR : Float;

  // [Removido: paintNow não utilizada]

  // Auxiliares de tempo/ORB/VRS
  rankVol                                             : Integer;
  isManha,isTarde,canFireORB,candleBuyOk,candleSellOk : Boolean;
  minutosDesdeAbertura,ORB_Minutos,orbMinutosEff      : Integer;
  kEff,orbStopRetesteATR_eff                          : Float;

  // Espelhos p/ nomes usados abaixo no código (compatibilidade)
  tpEff,slEff,medRangeDia                             : Float;
  cooldownBarsEff                                     : Integer;
  // === Controle de saída intrabar (histerese de confiança) ===
  confHi,confLo                                       : Float;

  // === Variáveis adicionadas (antes estavam no meio do código) ===
  ifrNorm,slopeNorm,vrsNorm                           : Float;
  janelaMin                                           : Integer;
  ratioThresh                                         : Float;
  marketMomentum                                      : Float;
  // Bollinger (séries corretas p/ permitir indexação; largura como snapshot numérico)
  bbTop,bbBot,bbMid                                   : Serie;
  bbWidth                                             : Float;
  gapBlock                                            : Boolean;
  tempoLimiteORB                                      : Integer;
  safeOffset,lagIdx                                   : Integer;
  // Janela dinâmica de pregão (adaptativa)
  horaInicioReal,horaFimReal                          : Integer;
  barrasDia,intervaloMin                              : Integer;
  // === Variáveis movidas do meio do código para o VAR global ===
  baseVRS,regimeAdj,volAdj                            : Float;
  minBodyDyn,wickMaxDyn                               : Float;
  confLive,confExit                                   : Float;
  tempoEstab                                          : Integer;
  holdBuy,holdSell,gateOpen                           : Boolean;

  // [REMOVIDO] bloco fantasma de TP/SL/qty não utilizado

  // Indicadores
  ifrValue                                            : Serie;
  media50                                             : Serie;
  media50Lag                                          : Float;

  // OTIMIZAÇÃO CIRÚRGICA (Item 6):
  // - slope NÃO precisa ser Série (não é indexado) → vira Float
  // - atrValue e vrs só são usados como snapshot atual → viram Float
  slope                                               : Float;
  slopeN                                              : Serie;

  atrValue                                            : Float;
  adxValue                                            : Serie;
  adxPrev                                             : Float;

  vrs                                                 : Float;

  ratioBW_ATR                                         : Float;
  vrsMinimo                                           : Float;
  // limiar dinâmico de VRS
  atrRelativo,atr0,vrs0                               : Float;

  // Estado de trade canônico
  statePrev                                           : Integer;
  barsDesdeSinal                                      : Integer;
  entryPrice,tp1Price,tp2Price,slPrice, highSinceEntry,lowSinceEntry : Float;
  parcialFeita                                        : Integer;

  lastEntrySide                                       : Integer;
  lastEntrySideExit                                   : Integer;  // <<< (CORREÇÃO) usado no AMARELO / DSS
  reentradasFeitas                                    : Integer;
  reentryArmed                                        : Integer;
  tSignal                                             : Integer;
  barsORB                                             : Integer;

  // Risco dinâmico
  TP1_eff,TP2_eff,SL_eff                              : Float;
  SL_Trail_FatorVol                                   : Float;
  // Thresholds adaptativos
  MinADX_eff,RatioBWATR_Min_eff,SlopeMin_eff          : Float;

  // Candle e registradores da barra anterior
  body,upperWick,lowerWick                            : Float;
  // Acrescenta variáveis usadas na detecção científica do Ponto Ótimo (compra/venda)
  // Mantém compatibilidade com todo o restante do script
  bodyATR,wickRatio,wickRatioCurrent,pnlCurrent,retrace5ATR,slope5N       : Float;
  doji,candleOk                                                           : Boolean;

  // Flag de decisão do Ponto Ótimo (AMARELO); precisa ser global (persistência determinística)
  optimalExit                                         : Boolean;
  // AMARELO soberano (nasce no “5m proxy” e NÃO pode ser cancelado por anti-ruído)
  forceExit5m                                         : Boolean;

  prevClose,prevHigh,prevLow,prevBbTop,prevBbBot      : Float;

  // VWAP/ORB/PDH-PDL/NR/Gap
  cumPV                                               : Float;
  cumV                                                : Float;
  vwapVal                                             : Float;
  orh,orl,orMid                                       : Float;
  orComputado                                         : Integer;
  pdh,pdl                                             : Float;
  nrTop,nrBot                                         : Float;
  trNow,trAvg                                         : Float;
  isNR                                                : Boolean;
  percentGap                                          : Float;
  gapAbertura                                         : Float;
  dataAtual                                           : Integer;
  dataAnterior                                        : Integer;
  // Registradores do DIA corrente (organismo vivo)
  dayHigh,dayLow,dayOpen,dayPrevClose                 : Float;
  isFirstBarOfDay                                     : Boolean;
  // Regime
  isTrend                                             : Boolean;
  regimeMercado                                       : Integer;
  // Gestão diária (execução real; preservado)
  tradesHoje                                          : Integer;
  lossesSeguidas                                      : Integer;
  cooling                                             : Integer;
  dRes                                                : Float;
  dResPeak                                            : Float;
  travaDia                                            : Boolean;
  travaGanho                                          : Boolean;

  // [REMOVIDO] qtdEff não utilizado

  // === ε dinâmicos usados na saída ótima (AMARELO) ===
  epsStep, epsRet                                     : Float;

  // === Camada 2 (Segurança Real) — variáveis auxiliares (precisam ser globais na NTSL) ===
  wick2Min                                            : Float;
  accelLoss                                           : Boolean;
  bbExtreme                                           : Boolean;
  noReversal                                          : Boolean;
  pnlATR2                                             : Float;

  // [REMOVIDO] parciais não implementadas neste script

  allowTrade                                          : Boolean;
  allowTradeGlobal                                    : Boolean;   // KILL-SWITCH GLOBAL (1 fonte de verdade)
  horarioPermitido                                    : Boolean;
  primeiraHora                                        : Boolean;

  // Sinais imediatos
  rompAlta1,rompBaixa1                                : Boolean;
  rompAlta1Vol,rompBaixa1Vol                          : Boolean;
  falsoAlta,falsoBaixa                                : Boolean;
  opBuySeed,opSellSeed                                : Boolean;
  // confirmação 2-steps
  // Normalização por barra (mutuamente exclusivos)
  buyCont,sellCont,buyFast,sellFast                   : Boolean;
  buyTrig,sellTrig                                    : Boolean;

  // Pintura (somente sinais) — enxuto e sem duplicidades
  // [REMOVIDO] prevState (variável não utilizada)
  // [REMOVIDO] paintBuy, paintSell (variáveis não utilizadas)
  // [REMOVIDO] allowSwitch (variável não utilizada)
  // Removido: paintState (redundante) e paintApply (estado “aplicado” é o próprio finalState)

  // ORÁCULO DINÂMICO — variáveis adaptativas (organismo vivo)
  regimeStrength                                      : Float;
  // 0..100 (força de tendência via ADX)
  volatilityState                                     : Integer;
  // 0=baixa | 1=média | 2=alta (ATR relativo)
  oracleConfidence                                    : Float;
  // 5..95 (% confiança para filtrar entradas)
  adaptiveBlock,adaptiveGateOk                        : Boolean;
  // bloqueio (congestão) e liberação (regime ok)
  buyStrength,sellStrength                            : Float;

  // Série persistente p/ máquina de estados (garante 1 cor por candle)
  paintStateSeries                                    : Serie;

  // 0=neutro | 1=compra(verde) | -1=venda(vermelho) | 2=saída(amarelo)
  // === ProfitGuardian (novas floats) ===
  PG_Trail_Fator, BreakEven_ATR_eff, atrMed, deltaOn, deltaOff,
  safetyBuffer, convictionBuy, convictionSell, lowRecent, highRecent : Float;

  // Snapshots de séries para uso seguro fora de condicionais/loops (Manual 4.5)
  vrsMedia0, atrMed0, adx0, slopeN0, close0, bbMid0   : Float;
  beArmed                                             : Boolean;

  // ===================== COLOR ENGINE Σ — FASE 1 / ESTABILIZAÇÃO =====================
  // finalState é a ÚNICA autoridade intrabar.
  // paintStateSeries[0] só recebe valor UMA VEZ, no commit final do candle.
  // Estados: -1=VENDA | 0=NEUTRO | 1=COMPRA | 2=SAÍDA/AMARELO.
  finalState                                          : Integer;
  rawBuy, rawSell                                     : Boolean;
  gateEntradaOk, gateSaidaOk                          : Boolean;

  // === Confiômetro e Gate do almoço (novas declarações) ===
  confADX, confSlope, conf        : Float;
  tp1Factor, tp2Factor            : Float;
  almocoJanela, rareOpportunity   : Boolean;
  distVWAP_BBmid_ATR              : Float;


  bothOn                          : Boolean;
  buyOn                           : Boolean;
  sellOn                          : Boolean;
  bias                            : Integer;

  stateLabel                      : String;

  // ===================== XAI / DSS / SA (núcleo simbólico) =====================
  // Objetivo: explicar "Motivo do Sinal" e "Motivo do Bloqueio" sem ML, sem custo,
  // 100% determinístico, compatível com NTSL/Profit.
  xaiSignalReason                 : String;  // motivo do VERDE/VERMELHO/AMARELO
  xaiBlockReason                  : String;  // motivo do NEUTRO (bloqueio)
  xaiContext                      : String;  // contexto: REGIME/VOL/HOR/GATE
  xaiRegimeLabel                  : String;
  xaiVolLabel                     : String;

  xaiExitCause                    : String;  // CAUSA REAL do AMARELO (decisão final de saída)
  xaiGuardNote                    : String;  // NOTA do ProfitGuardian (BE/Trail/TimeStop) — NÃO é saída
  exitByConfidence                : Boolean; // telemetria (não decide)
  exitByReversal                  : Boolean; // telemetria (não decide)
  exitByStop                      : Boolean; // telemetria (não decide)


  // ===================== TRADE AUDITOR (determinístico, leve e robusto) =====================
  // Núcleo: registra qualidade do amarelo (trade fechado) + telemetria adaptativa (EWMA) + proteção (podas)
  // Sem funções extras. Sem ML. Sem custo alto. 100% determinístico.
  auditExitBar                     : Integer;
  auditScore                       : Integer;
  auditTrades                      : Integer;

  auditExitPx                      : Float;
  auditEntryPx                     : Float;
  auditMFE                         : Float;
  auditMAE                         : Float;
  auditPnL                         : Float;
  auditPnL_ATR                     : Float;
  auditEfficiency                  : Float;

  auditEffNorm                     : Float;
  auditPnLNorm                     : Float;
  auditPainNorm                    : Float;

  auditExitReason                  : String;
  auditEntryReason                 : String;
  auditTag                         : String;

  // --- Métricas adaptativas (EWMA) — organismo vivo (online, leve) ---
  auditAlpha                       : Float;   // 0.05..0.25 (dinâmico por vol/impulso)
  auditEwmaWin                     : Float;   // 0..1
  auditEwmaGain_ATR                : Float;   // >=0
  auditEwmaLoss_ATR                : Float;   // >=0
  auditExpectancy_ATR              : Float;   // ganho - perda (em ATR)
  auditProfitFactor                : Float;   // ganho/perda
  auditEwmaEff                     : Float;   // 0..1

  // --- Curva & Drawdown em ATR (leve, determinístico) ---
  auditEquity_ATR                  : Float;
  auditPeak_ATR                    : Float;
  auditDD_ATR                      : Float;
  auditMaxDD_ATR                   : Float;

  // --- Proteção dinâmica (podas inteligentes, sem quebrar cores) ---
  auditProtActive                  : Boolean;
  auditProtBarsLeft                : Integer;
  auditProtReason                  : String;
  gateAuditorOk                    : Boolean;

  // --- Supervisor Walk-Forward / Fora da Amostra / Anti-Overfitting ---
  wfCycleBars                      : Integer;
  wfPosInCycle                     : Integer;
  wfFoldId                         : Integer;
  wfTrainTrades                    : Integer;
  wfOOSTrades                      : Integer;
  wfIsTrain                        : Boolean;
  wfIsOOS                          : Boolean;
  wfGateOk                         : Boolean;
  wfPenalty                        : Float;
  wfTrainWins                      : Float;
  wfOOSWins                        : Float;
  wfTrainWinRate                   : Float;
  wfOOSWinRate                     : Float;
  wfTrainGain_ATR                  : Float;
  wfTrainLoss_ATR                  : Float;
  wfOOSGain_ATR                    : Float;
  wfOOSLoss_ATR                    : Float;
  
  wfTrainExp_ATR                   : Float;
  wfOOSExp_ATR                     : Float;
  wfTrainPF                        : Float;
  wfOOSPF                          : Float;
  wfTrainEquity_ATR                : Float;
  wfTrainPeak_ATR                  : Float;
  wfTrainDD_ATR                    : Float;
  wfTrainMaxDD_ATR                 : Float;
  wfOOSEquity_ATR                  : Float;
  wfOOSPeak_ATR                    : Float;
  wfOOSDD_ATR                      : Float;
  wfOOSMaxDD_ATR                   : Float;
  wfBlockReason                    : String;

  // --- Execução real / custos / fill confirmado ---
  execCostTicks                    : Float;
  execCostTicksEff                 : Float;
  execCostPx                       : Float;
  auditPnLBruto                    : Float;
  costFloorATR                     : Float;
  costFloorPx                      : Float;

  // sizing efetivo por barra (executável)
  qtyEntryEff                      : Integer;
  qtyExitEff                       : Float;

  exitPending                      : Boolean;
  exitPendingSide                  : Integer;
  exitSignalBar                    : Integer;

  // --- Contrato formal de ENTRADA: evita duplicar ordem enquanto há ordem pendente ---
  entryPending                     : Boolean;
  entryPendingSide                 : Integer;
  entrySignalBar                   : Integer;

  // --- Registradores de sequência para o Color Engine (retro-pintura determinística) ---
  hiBarIdx, loBarIdx, entryBarIdx, resetBars : Integer;

  offsetMaxima                    : Integer;
  offsetMinima                    : Integer;
  infoText                        : String;

  maxGainBuy, maxGainSell         : Float;
  entropySignal, regimeAdjExit    : Float;
  confidenceAmarelo               : Boolean;

  // === (FIX COMPILAÇÃO) requisitos dinâmicos do gate + reversão com lucro mínimo ===
  vrsReq                          : Float;
  confReq                         : Float;
  minProfitATR                    : Float;
  currentProfitATR                : Float;

  // === VARIÁVEIS DO ORÁCULO QUÂNTICO ===
  currentHigh, currentLow, currentBody, currentRange             : Float;
  maxGainATR, currentGainATR, upperWickCurrent, lowerWickCurrent : Float;
  condition1, condition2, condition3, condition4, condition5     : Boolean;
  confidenceScore, confidenceThreshold                           : Float;

  // === Autoajuste supervisionado + semântica intrabar do stop ===
  minGainTarget, epsStepTarget, epsRetTarget, adaptAlpha, stopTouchPx : Float;
  stopTouchedBar                                                  : Boolean;

  // === Governança WF/OOS + produção científica ===
  wfTrainScore, wfOOSScore, wfOverfitGap, wfOverfitScore,
  wfStabilityScore, wfPromotionScore, wfEdgeRatio                 : Float;
  wfFoldApproved, wfPromotionOk, wfOverfitOk, wfOOSWarmupOk       : Boolean;
  wfConsecApproved, wfConsecRejected                              : Integer;
  prodReady, prodBlock                                            : Boolean;
  prodReason                                                      : String;

  // === Contrato AMARELO visual ↔ fill real + captura do ponto ótimo ===
  yellowTheoreticalPx, yellowExtremePx, yellowVisualPx, yellowRealPx : Float;
  yellowCaptureEff, yellowGivebackATR, yellowNetATR, yellowFillDriftTicks : Float;
  yellowCaptureOk                                                 : Boolean;
  execPendingBars, entryPendingBars, exitPendingBars              : Integer;
  stopOffsetTicksEff, gapRiskATR, execNetEdgeATR                  : Float;
  extremeGapActive                                                : Boolean;

  // === Suíte de regressão pétrea automática ===
  regFailCount                                                    : Integer;
  regSeenBuyTopRev, regSeenSellBottomRev, regSeenStopVol          : Boolean;

  // [REMOVIDO] lastEntrySide / lastEntrySideExit redeclarados (já existem no VAR principal)

  atrSerie                        : Serie;

////////////////////////////// INÍCIO ///////////////////////////////////////
begin
  // ===== INICIALIZAÇÃO DO ORGANISMO VIVO =====
  if CurrentBar() = 0 then
    begin
      // === Estado do Sistema ===
      statePrev := 0;
      barsDesdeSinal := 0;
      parcialFeita := 0;
      lastEntrySide := 0;
      lastEntrySideExit := 0;
      reentradasFeitas := 0;
      reentryArmed := 0;
      tSignal := 0;
      // === ORB e VWAP ===
      cumPV := 0;
      cumV := 0;
      vwapVal := 0;
      orh := - 999999;
      orl := 999999;
      orMid := 0;
      orComputado := 0;
      barsORB := 0;
      // === Registradores do Organismo (Dia Anterior) ===
      pdh := 0;
      // PDH (dia anterior) - será preenchido no 1º troca-de-dia
      pdl := 0;
      // PDL (dia anterior)
      // === Estado do Dia Corrente ===
      percentGap := 0;
      // % de gap de abertura do dia corrente
      gapAbertura := 0;
      // gap absoluto do dia corrente
      dayHigh := High[0];
      dayLow  := Low[0];
      dayOpen := Open[0];

      // abertura do dia corrente
      // Correção-Σ:
      // No CurrentBar() = 0, prevClose ainda não foi consolidado pelo bloco safeOffset.
      // Portanto, o organismo nasce sem gap artificial.
      dayPrevClose := Close[0];
      isFirstBarOfDay := True;

      // marcado até processar o 1º candle do novo dia
      // === Gestão de Risco Diário ===
      tradesHoje := 0;
      lossesSeguidas := 0;
      cooling := 0;
      dRes := 0;
      dResPeak := 0;
      travaDia := False;
      travaGanho := False;

      // === Estado de Pintura (CORE) — Série canônica (sem indexador na escrita) ===
      // === INICIALIZAÇÃO CIENTÍFICA DO ORÁCULO - ORGANISMO VIVO ===
      // Sistema baseado em física de partículas financeiras (Manual NTSL 4.1-4.7)
      // [REMOVIDO] prevState — variável legada. Estado canônico é paintStateSeries/statePrev.

      // Inicialização canônica da série de estado (Manual 4.5 - Séries de dados)
      paintStateSeries[0] := 0; // NEUTRO canônico (SÉRIE) na barra atual

       // === REGISTRADORES DO CICLO VITAL ===
       hiBarIdx := 0;      // Índice do candle de máxima (compra)
       loBarIdx := 0;      // Índice do candle de mínima (venda)
       entryBarIdx := 0;   // Índice da entrada
       resetBars := 0;     // Contador de reset neutro

       // === ESTADO INICIAL DO AUDITOR DE TRADE ===
       auditTrades := 0;   // Contador de trades fechados (amarelos)
       auditScore  := 0;   // Score de qualidade inicial
       auditEquity_ATR := 0.0;  // PnL acumulado em ATR
       auditMaxDD_ATR  := 0.0;  // Drawdown máximo em ATR
       auditPeak_ATR   := 0.0;
       auditDD_ATR     := 0.0;
       auditEwmaWin    := 0.0;
       auditEwmaGain_ATR := 0.0;
       auditEwmaLoss_ATR := 0.0;
       auditExpectancy_ATR := 0.0;
       auditProfitFactor   := 0.0;
       auditEwmaEff        := 0.0;
       auditProtActive     := False;
       auditProtBarsLeft   := 0;
       auditProtReason     := "";
       gateAuditorOk       := True;

       // === ESTADO INICIAL DO SUPERVISOR WALK-FORWARD / OOS ===
       wfCycleBars       := Max(2, WF_BarrasTreino + WF_BarrasOOS);
       wfPosInCycle      := 0;
       wfFoldId          := 0;
       wfTrainTrades     := 0;
       wfOOSTrades       := 0;
       wfIsTrain         := True;
       wfIsOOS           := False;
       wfGateOk          := True;
       wfPenalty         := 0.0;
       wfTrainWins       := 0.0;
       wfOOSWins         := 0.0;
       wfTrainWinRate    := 0.0;
       wfOOSWinRate      := 0.0;
       wfTrainGain_ATR   := 0.0;
       wfTrainLoss_ATR   := 0.0;
       wfOOSGain_ATR     := 0.0;
       wfOOSLoss_ATR     := 0.0;
       wfTrainExp_ATR    := 0.0;
       wfOOSExp_ATR      := 0.0;
       wfTrainPF         := 999.0;
       wfOOSPF           := 999.0;
       wfTrainEquity_ATR := 0.0;
       wfTrainPeak_ATR   := 0.0;
       wfTrainDD_ATR     := 0.0;
       wfTrainMaxDD_ATR  := 0.0;
       wfOOSEquity_ATR   := 0.0;
       wfOOSPeak_ATR     := 0.0;
       wfOOSDD_ATR       := 0.0;
       wfOOSMaxDD_ATR    := 0.0;
       wfBlockReason     := "";

       // === ESTADOS DE PERSISTÊNCIA ORACULAR ===
       holdBuy := False;   // Compra ativa
       holdSell := False;  // Venda ativa
       beArmed := False;   // Breakeven armado

       exitPending := False;
       exitPendingSide := 0;
       exitSignalBar := -1;

       entryPending := False;
       entryPendingSide := 0;
       entrySignalBar := -1;

       execCostTicks := 0.0;
       execCostPx := 0.0;
       auditPnLBruto := 0.0;

       // === Governança WF/OOS + produção ===
       wfTrainScore      := 0.0;
       wfOOSScore        := 0.0;
       wfOverfitGap      := 0.0;
       wfOverfitScore    := 1.0;
       wfStabilityScore  := 0.0;
       wfPromotionScore  := 0.0;
       wfEdgeRatio       := 0.0;
       wfFoldApproved    := False;
       wfPromotionOk     := False;
       wfOverfitOk       := True;
       wfOOSWarmupOk     := False;
       wfConsecApproved  := 0;
       wfConsecRejected  := 0;
       prodReady         := False;
       prodBlock         := False;
       prodReason        := "PROD=SEM_WF";

       // === AMARELO ↔ Fill real + captura ===
       yellowTheoreticalPx := 0.0;
       yellowExtremePx     := 0.0;
       yellowVisualPx      := 0.0;
       yellowRealPx        := 0.0;
       yellowCaptureEff    := 0.0;
       yellowGivebackATR   := 0.0;
       yellowNetATR        := 0.0;
       yellowFillDriftTicks := 0.0;
       yellowCaptureOk     := True;
       execPendingBars     := 0;
       entryPendingBars    := 0;
       exitPendingBars     := 0;
       stopOffsetTicksEff  := 0.0;
       gapRiskATR          := 0.0;
       execNetEdgeATR      := 0.0;
       extremeGapActive    := False;

      // === Estados de Trade ===
      entryPrice := 0;
      tp1Price := 0;
      tp2Price := 0;
      slPrice := 0;
      stopTouchPx := 0;
      highSinceEntry := 0;
      lowSinceEntry  := 0;
      minGainTarget := 0;
      epsStepTarget := 0;
      epsRetTarget := 0;
      adaptAlpha := 0;
      stopTouchedBar := False;
      regFailCount := 0;
      regSeenBuyTopRev := False;
      regSeenSellBottomRev := False;
      regSeenStopVol := False;

      // === Estados Adaptativos do Organismo ===
      regimeStrength := 0;
      volatilityState := 1;
      // Inicia como volatilidade média
      oracleConfidence := 50;
      // Confiança inicial moderada
      adaptiveBlock := False;
      adaptiveGateOk := True;
      buyStrength := 0;
      sellStrength := 0;
      // === Estados de Persistência ===
      // [Removido: segunda atribuição redundante de holdBuy/holdSell]

      // === Debug Inicial ===
      if ModoDebug then
        PlotText("ORG_NASC", clAqua, 1, 12, High[0] + (High[0] - Low[0]));
    end;

  // BB canônicas (Manual): mid/top/bot como Séries determinísticas, sem operadores inválidos
  bbMid := Media(Periodo, Close);
  bbTop := bbMid + (Desvio * StdDevConstPeriod(Close, Periodo));
  bbBot := bbMid - (Desvio * StdDevConstPeriod(Close, Periodo));

  // largura BB como Float da barra atual — indexação aplicada ao resultado (tipo Float garantido)
  bbWidth := Max(0.0, bbTop[0] - bbBot[0]);

 // --- PRÉ-CÁLCULO SEGURO DOS REGISTRADORES (SEM ACESSO POSICIONAL EM IF/LOOP) ---
  safeOffset := 0;
  if CurrentBar() > 0 then
    safeOffset := 1;
  prevClose := Close[safeOffset];
  prevHigh := High[safeOffset];
  prevLow := Low[safeOffset];
  prevBbTop := bbTop[safeOffset];
  prevBbBot := bbBot[safeOffset];

  // RSI (IFR) conforme Manual — Tipo=0 (Clássico)
  ifrValue := RSI(PeriodoIFR,0);
  media50 := Media(ConfirmacaoTendencia,Close);

  // --- LAG SEGURO (Manual: primeiro candle pode ter [0] "zerado"/instável) ---
  // Regra: lagIdx >= 1 quando a intenção é "lag"; no bar 0 não existe passado.
  if CurrentBar() <= 0 then
  begin
    lagIdx := 0;
    media50Lag := media50[0];
    slope := 0.0;                 // neutro científico: evita ruído no nascimento do organismo
  end
  else
  begin
    // clamp: mínimo 1 para garantir passado real; máximo PeriodoInclinacao ou CurrentBar()
    lagIdx := Min(PeriodoInclinacao, CurrentBar());
    if lagIdx < 1 then lagIdx := 1;

    media50Lag := media50[lagIdx];
    slope := media50 - media50Lag;
  end;

  // OTIMIZAÇÃO CIRÚRGICA (Item 6):
  // atrValue agora é Float (snapshot do ATR do candle atual).
  atrSerie := AvgTrueRange(ATR_Periodo,2);
  atrValue := atrSerie[0];

  // Snapshot seguro (Float) para a barra atual, com fallback mínimo:
  atr0 := Max(atrValue, Max(High[0] - Low[0], 0.0001));

  // Normalização robusta do slope: usa o snapshot seguro do ATR (atr0) para evitar divisão por zero  
  slopeN := slope / atr0;

  adxValue := ADX(ADX_Periodo,ADX_Media);
  // Snapshot do ADX anterior com blindagem de bootstrap
  if CurrentBar() > 0 then
    adxPrev := adxValue[1]
  else
    adxPrev := adxValue[0];

  // === SNAPSHOTS POSICIONAIS — fora de condicionais/loops (Manual 4.5) ===
  adx0    := adxValue[0];
  slopeN0 := slopeN[0];
  close0  := Close[0];
  bbMid0  := bbMid[0];

  // VRS canônico da barra: 1 fonte de verdade + proteção determinística
  vrsMedia0 := Max(0.0001, Media(VRS_Periodo, Volume)[0]);
  vrs       := Volume[0] / vrsMedia0;
  vrs0      := Max(0.0, vrs);

  // ==========================================================
  // SINCRONIZAÇÃO COM POSIÇÃO REAL (SOBERANA EM EXECUÇÃO)
  // Regras:
  // - Em SomenteSinais=TRUE, ColorirPorPosicao continua opcional.
  // - Em SomenteSinais=FALSE, a posição real SEMPRE prevalece.
  // - Isso evita AMARELO visual sem fill real e elimina drift entre
  //   lógica interna, auditoria, stop e automação.
  // ==========================================================
  if (not SomenteSinais) or (ColorirPorPosicao and HasPosition) then
  begin
    if HasPosition then
    begin
      // ==========================================================
      // SINCRONIZAÇÃO REAL DO LADO DA SAÍDA — Correção-Σ
      // Em execução real, a posição real é soberana.
      // Em SomenteSinais, só espelha posição real se ela existir.
      // Nunca zera hold lógico apenas por ausência de posição real.
      // ==========================================================
      if IsBought then
      begin
        holdBuy           := True;
        holdSell          := False;
        lastEntrySide     := 1;
        lastEntrySideExit := 1;
      end
      else if IsSold then
      begin
        holdSell          := True;
        holdBuy           := False;
        lastEntrySide     := -1;
        lastEntrySideExit := -1;
      end;

      entryPrice := Price;

      if (entryPrice <= 0) then
      begin
        if IsBought then
          entryPrice := BuyPrice
        else if IsSold then
          entryPrice := SellPrice
        else
          entryPrice := Close[0];
      end;

      if (highSinceEntry <= 0) then highSinceEntry := High[0];
      if (lowSinceEntry  <= 0) then lowSinceEntry  := Low[0];

      if High[0] > highSinceEntry then highSinceEntry := High[0];
      if Low[0]  < lowSinceEntry  then lowSinceEntry  := Low[0];

      if (slPrice <= 0) then
      begin
        if IsBought then
          slPrice := entryPrice - (SL_Mult * atr0)
        else
          slPrice := entryPrice + (SL_Mult * atr0);
      end;
    end
    else
    begin
      // Só a execução real pode zerar estado lógico por ausência de posição.
      // No modo SomenteSinais, a máquina de cores é soberana.
      if (not SomenteSinais) then
      begin
        holdBuy   := False;
        holdSell  := False;

        if not exitPending then
        begin
          lastEntrySide     := 0;
          lastEntrySideExit := 0;
        end;

        if exitPending then
        begin
          resetBars       := 1;
          exitPending     := False;
          exitPendingSide := 0;
          exitSignalBar   := -1;
          slPrice         := 0;
          highSinceEntry  := 0;
          lowSinceEntry   := 0;
        end;
      end;
    end;
  end;

  // ==========================================================
  // CONSOLIDA PRIMEIRO VOLATILIDADE / REGIME DA BARRA ATUAL
  // E SÓ DEPOIS APLICA O PROFITGUARDIAN
  // ==========================================================

  // Snapshot do VRS atual para cálculos
  // === Auxiliares dinâmicos ausentes (janela, VRS e ORB) ===
  // Minutos desde a 1ª barra real do dia
  // Janela real de pregão (conforme inputs; fallback B3 seguro)
  // Fonte única de verdade da janela real:
  // - horaInicioReal nasce na 1ª barra real do dia
  // - horaFimReal respeita HoraFim no bloco canônico da sessão
  // Não sobrescrever aqui para não distorcer:
  // minutosDesdeAbertura, tempoLimiteORB e horarioPermitido

  // Rank de volatilidade por ATR relativo (robusto a zeros)
  // ATR relativo (1 fonte de verdade): NÃO recalcular atrSerie aqui (já calculado acima)
  atrMed0     := Media(ATR_Periodo, atrSerie)[0];
  atrMed      := Max(0.0001, Max(atr0, atrMed0));
  atrRelativo := atr0 / atrMed;
  rankVol := Min(10,Max(0,Round(atrRelativo * 10)));

  // Ganho efetivo do ORB
  kEff := ORB_K;

  // ORB adaptativo (clamps finos por regime/volatilidade)
  // BLINDAGEM REAL: consolida thresholds efetivos ANTES do regime, usando rankVol da barra atual.
  if (rankVol >= 7) then
    volatilityState := 2
  else if (rankVol >= 4) then
    volatilityState := 1
  else
    volatilityState := 0;

  MinADX_eff := MinADX;
  RatioBWATR_Min_eff := RatioBWATR_Min;
  SlopeMin_eff := SlopeMin;

  if volatilityState = 0 then
  begin
    MinADX_eff := MinADX + 4.0;
    RatioBWATR_Min_eff := RatioBWATR_Min + 0.06;
    SlopeMin_eff := Max(0.00,SlopeMin + 0.06);
  end
  else if volatilityState = 1 then
  begin
    MinADX_eff := MinADX + 2.0;
    RatioBWATR_Min_eff := RatioBWATR_Min + 0.03;
    SlopeMin_eff := Max(0.00,SlopeMin + 0.03);
  end;

  // Largura BB normalizada por ATR snapshot (robusta a zero/ruído)
  // PRECISA vir ANTES do regime para evitar atraso lógico na barra atual
  if (atr0 > 0.0001) then
    ratioBW_ATR := bbWidth / atr0
  else
    ratioBW_ATR := 0;

  if RegimeAware then
  begin
    if (adx0 >= MinADX_eff) then
      regimeMercado := 0    
    else if (ratioBW_ATR < RatioBWATR_Min_eff) then
      regimeMercado := 3    
    else if (Abs(slopeN0) >= SlopeMin_eff) then    
      regimeMercado := 2
    else
      regimeMercado := 1;
  end
  else
  begin

    // Default determinístico quando RegimeAware=OFF
    regimeMercado := 1;
  end;

  // ===== PROFITGUARDIAN DINÂMICO — AGORA NO LUGAR CORRETO =====
  // Depois de volatilityState e regimeMercado estarem atualizados na barra atual
  if HasPosition or holdBuy or holdSell then
    begin
      // 1) Direção e snapshots (imutáveis por barra)
      if HasPosition then
      begin
        if IsBought then
          dir := 1
        else
          dir := -1;
      end
      else if holdBuy then
        dir := 1
      else
        dir := -1;

      moveATR := Max(atr0, 0.0001);                                 // snapshot seguro do ATR
      pnlATR  := ((close0 - entryPrice) * dir) / moveATR;

      // BE verdadeiro = preço de entrada (sem colchão)
      bePrice   := entryPrice;
      beCushion := entryPrice + (dir * PG_ATR_Breakeven_Offset * moveATR);

      // 2) Base canônica do stop — sem depender de SL_eff antes da consolidação da barra
      // Regra:
      // - se já existe slPrice válido, ele é a fonte soberana;
      // - se ainda não existe, usa fallback estável e determinístico via SL_Mult;
      if (slPrice > 0) then
        slCandidate := slPrice
      else
      begin
        if (dir = 1) then
          slCandidate := entryPrice - (SL_Mult * moveATR)
        else
          slCandidate := entryPrice + (SL_Mult * moveATR);
      end;

      // 3) Breakeven com histerese
      if (volatilityState = 0) then
      begin
        deltaOn  := 0.02;
        deltaOff := 0.01;
      end
      else if (volatilityState = 2) then
      begin
        deltaOn  := 0.08;
        deltaOff := 0.04;
      end
      else
      begin
        deltaOn  := 0.05;
        deltaOff := 0.03;
      end;

      // ==========================================================
      // BREAKEVEN DIRECIONAL — Correção-Σ
      // Só arma proteção quando existe lucro real em ATR.
      // Nunca usar Abs(pnlATR), pois prejuízo não é lucro.
      // ==========================================================
      if (not beArmed) and (pnlATR >= (PG_ATR_Breakeven_Trigger + deltaOn)) then
        beArmed := True;

      if (beArmed) and (pnlATR <= (PG_ATR_Breakeven_Trigger - deltaOff)) then
        beArmed := False;

      if beArmed then
      begin
        if (dir = 1) then
          slCandidate := Max(slCandidate, beCushion)
        else
          slCandidate := Min(slCandidate, beCushion);
      end;

      if (barsDesdeSinal >= PG_TimeStop_Bars) and
         (pnlATR >= 0) and
         (pnlATR < PG_TimeStop_MinATR) then
      begin
        if (dir = 1) then
          slCandidate := Max(slCandidate, bePrice)
        else
          slCandidate := Min(slCandidate, bePrice);

        if (xaiGuardNote = "") then xaiGuardNote := "PG=TIMESTOP_BE";
      end;

      // 5.5) TRAILING STOP POR ATR (ATIVA O INPUT Trail_ATR_Fator)
      // Objetivo: travar lucro progressivamente sem depender 100% do AMARELO.
      // Regra: trailing só APERTA (nunca afrouxa), com clamps por regime/volatilidade.
      trailDist := Trail_ATR_Fator;

      // Se Trail_ATR_Fator <= 0, trailing fica OFF (zero efeitos colaterais).
      if (trailDist > 0) then
      begin
        // Autoajuste leve (organismo vivo) — agora usando o estado ATUAL da barra
        if RiskAdapt then
        begin
          if (volatilityState = 2) then
            trailDist := trailDist + 0.10
          else if (volatilityState = 0) then
            trailDist := trailDist - 0.05;

          if (regimeMercado = 3) then
            trailDist := trailDist + 0.05;
        end;

        // Clamps científicos
        trailDist := Max(0.20, Min(1.30, trailDist));

        // Garante extremos para trailing mesmo em modo posição real
        if (dir = 1) and (High[0] > highSinceEntry) then
          highSinceEntry := High[0]
        else if (dir = -1) and (Low[0] < lowSinceEntry) then
          lowSinceEntry := Low[0];

        if (dir = 1) then
        begin
          trailBase   := highSinceEntry - (trailDist * moveATR);
          slCandidate := Max(slCandidate, trailBase);
        end
        else
        begin
          trailBase   := lowSinceEntry + (trailDist * moveATR);
          slCandidate := Min(slCandidate, trailBase);
        end;

        if (xaiGuardNote = "") then xaiGuardNote := "PG=TRAIL_ATR";
      end;

      // 6) Clamps científicos
      // ==========================================================
      // CLAMP ANTI-AFROUXAMENTO — Correção-Σ
      // Compra: stop nunca pode descer.
      // Venda: stop nunca pode subir.
      // ==========================================================
      if (dir = 1) then
      begin
        candidateCeil := close0 - (0.05 * moveATR);

        if (slPrice > 0) then
          slCandidate := Max(slPrice, Min(slCandidate, candidateCeil))
        else
          slCandidate := Min(slCandidate, candidateCeil);
      end
      else
      begin
        candidateFloor := close0 + (0.05 * moveATR);

        if (slPrice > 0) then
          slCandidate := Min(slPrice, Max(slCandidate, candidateFloor))
        else
          slCandidate := Max(slCandidate, candidateFloor);
      end;

      slPrice := slCandidate;
    end;

  orbMinutosEff := ORB_MinClamp + Round(((ORB_MaxClamp - ORB_MinClamp) * rankVol) / Max(kEff,0.0001));

  // micro-ajuste por regime: tendência encurta; inclinação/congestão alonga (respeitando limites)
  if (regimeMercado = 0) then
    orbMinutosEff := Max(ORB_MinClamp, orbMinutosEff - 2)
  else if (regimeMercado = 2) then
    orbMinutosEff := Min(ORB_MaxClamp, orbMinutosEff + 2)
  else if (regimeMercado = 3) then
    orbMinutosEff := Min(ORB_MaxClamp, orbMinutosEff + 4);

  ORB_Minutos := Min(ORB_MaxClamp, Max(ORB_MinClamp, orbMinutosEff));

  // === Reteste obrigatório em baixa/média liquidez (rankVol ≤ 4)
  // Eleva o raio do reteste em vol. baixa: 0.8–1.0 ATR
  orbStopRetesteATR_eff := ORB_StopReteste_ATR;
  if ORB_PermiteReteste then
    begin
      if (rankVol <= 4) then
        orbStopRetesteATR_eff := Max(0.80, orbStopRetesteATR_eff);
      if (volatilityState = 0) then
        orbStopRetesteATR_eff := Min(1.00, Max(0.90, orbStopRetesteATR_eff));
    end;

  // Média de range do dia (proxy robusto)
  medRangeDia := atr0;

  // Critério canônico de sessão:
  // manhã/tarde por horário fixo em HHMM, conforme Manual NTSL
  isManha := (Time < 1200);
  isTarde := not isManha;

  // [SUBSTITUÍDO] vrsMinimo é calculado no bloco "VRS mínimo dinâmico" (sem duplicidade).

  // Removido espelho prematuro: tpEff/slEff serão definidos nos blocos VRS e RiskAdapt
  // (mantém a coerência de fluxo e evita uso de variáveis ainda não inicializadas)

  // ===== Horário / resets =====
  // hora removida (não utilizada); Time é usado diretamente nos filtros conforme o Manual.
  dataAtual := Date;
  dataAnterior := Date[safeOffset];

  // ===== ORGANISMO VIVO: Detecção de Troca de Dia =====
  if (dataAtual <> dataAnterior) then
    begin
      // Promove extremos do dia anterior para PDH/PDL "congelados"
      if (dayHigh > 0) and (dayLow > 0) then
        begin
          pdh := dayHigh;
          pdl := dayLow;
        end;

      // Guarda o último fechamento do dia anterior para gap
      dayPrevClose := prevClose;

      // ==========================================================
      // RENASCIMENTO DIÁRIO (DETERMINÍSTICO) — SEM CARRY-OVER
      // ==========================================================

      // Preparação do novo dia - ORGANISMO RENASCE
      dayHigh := High;
      dayLow := Low;
      dayOpen := Open;
      isFirstBarOfDay := True;

      // Reset de pausa com preservação pétrea do NEUTRO pós-AMARELO.
      // Se ontem terminou em AMARELO, o primeiro candle útil do novo dia
      // ainda precisa respeitar 1 candle neutro obrigatório.
      if (CurrentBar() > 0) and ((statePrev = 2) or (paintStateSeries[1] = 2)) then
        resetBars := Max(resetBars, 1)
      else
        resetBars := 0;

      // Reset do organismo para novo ciclo
      tradesHoje := 0;

      // Cooling: novo dia = nova oportunidade (evita “morrer travado”)
      cooling := 0;

      // Resultado diário: ZERA (StopDia precisa ser do DIA ATUAL)
      dRes := 0;
      dResPeak := 0;

      // Travas diárias: ZERA no nascimento (evita nascer travado)
      travaGanho := False;
      travaDia := False;

      // Reset de estado de trade:
      // só zera completamente quando NÃO houver posição real.
      optimalExit := False;

      if not HasPosition then
      begin
        holdBuy := False;
        holdSell := False;
        lastEntrySide := 0;
        lastEntrySideExit := 0;
        slPrice := 0;
        beArmed := False;
        stopTouchPx := 0;
        stopTouchedBar := False;
      end
      else if IsBought then
      begin
        holdBuy := True;
        holdSell := False;
        lastEntrySide := 1;
      end
      else if IsSold then
      begin
        holdSell := True;
        holdBuy := False;
        lastEntrySide := -1;
      end;

      // ===== RESET DO AUDITOR (NOVO DIA = NOVA SÉRIE) =====
      auditTrades := 0;
      auditEquity_ATR := 0.0;
      auditPeak_ATR := 0.0;
      auditDD_ATR := 0.0;
      auditMaxDD_ATR := 0.0;
      auditProtActive := False;
      auditProtBarsLeft := 0;
      auditProtReason := "";

      // ===== RESET DA SUÍTE DE REGRESSÃO PÉTREA =====
      // (removidas reinicializações que interrompiam o acúmulo intrabar)
      // regFailCount := 0;
      // regSeenBuyTopRev := False;
      // regSeenSellBottomRev := False;
      // regSeenStopVol := False;

      // ===== RESET DE MARCADORES INTRABAR =====
      stopTouchedBar := False;
      stopTouchPx := 0;

      // ORB reset - novo ciclo
      orh := High;
      orl := Low;
      orComputado := 0;
      barsORB := 0;
    end;


  // ===== ORGANISMO VIVO: Processamento do Primeiro Candle do Dia =====
  if (isFirstBarOfDay) then
    begin
      dayOpen := Open;

      // Cálculo do gap com proteção contra divisão por zero
      if (dayPrevClose <> 0) then
        begin
          gapAbertura := dayOpen - dayPrevClose;
          percentGap := 100.0 * Abs(gapAbertura) / Max(0.0001,dayPrevClose);
        end
      else
        begin
          gapAbertura := 0;
          percentGap := 0;
        end;

      // Primeiro candle do dia nasce neutro
      paintStateSeries[0] := 0;
    end;

  // RESET NEUTRO PÓS-AMARELO
  // Mantido EXCLUSIVAMENTE no bloco canônico do Color Engine.
  // Aqui não forçamos estado para evitar duplicidade de autoridade.

  // ===== ATUALIZAÇÃO DO ORGANISMO VIVO: Extremos do Dia Corrente =====
  if (High > dayHigh) then
    dayHigh := High;
  if (Low < dayLow) then
    dayLow := Low;

  // ===== Janela dinâmica de pregão =====
  // Hora inicial real = hora da 1ª barra do dia
  if (dataAtual <> dataAnterior) or isFirstBarOfDay then
    begin
      horaInicioReal := Time;
      // 1ª barra do dia
      // Estimação do fim: nº de barras do dia x intervalo (em minutos)
      if (HoraFim > 0) then
        horaFimReal := HoraFim     // respeita janela definida nos inputs/Manual
      else
        horaFimReal := 1800;       // CLAMP seguro em HHMM, conforme Manual NTSL
    end;

  // Agora sim a referência da abertura está correta para a barra atual
  minutosDesdeAbertura := Max(0,TimeToMinutes(Time) - TimeToMinutes(horaInicioReal));

  // Só depois de usar a flag no bootstrap da sessão ela é consumida
  if (isFirstBarOfDay) then
    isFirstBarOfDay := False;

  // Fim da Opening Range (ORB) calculado a partir do início real do dia (1 fonte de verdade)
  tempoLimiteORB := MinutesToTime(TimeToMinutes(horaInicioReal) + ORB_Minutos);

  // Sempre respeitar a janela dinâmica (mesmo com FiltroHorario desligado)
  horarioPermitido := (Time >= horaInicioReal) and (Time < horaFimReal);

  // Opcionais: restrições adicionais do usuário, se ativas
  if FiltroHorario then
    horarioPermitido := horarioPermitido and (Time >= HoraInicio) and (Time < HoraFim);

  // Bloqueia trades na 1ª hora, se selecionado
  if (BloquearPrimeiraHora and (Time < MinutesToTime(TimeToMinutes(horaInicioReal) + 60))) then
    horarioPermitido := False;

  // 1ª hora real do dia
  // ===== Reset de sinais por barra =====
  // Cooling do organismo: decrementa 1 por candle (CooldownLoss_Bars em barras)
  if (cooling > 0) then
    cooling := cooling - 1;

  buyCont := False;
  sellCont := False;
  buyFast := False;
  sellFast := False;
  tSignal := 0;
  // (não estritamente necessário, mas explicitamos para clareza)

  rompAlta1 := False;
  rompBaixa1 := False;
  rompAlta1Vol := False;
  rompBaixa1Vol := False;
  falsoAlta := False;
  falsoBaixa := False;
  opBuySeed := False;
  opSellSeed := False;

  // [REMOVIDO] allowSwitch — variável legada. Gates já são: allowTrade, adaptiveGateOk, candleOk, horarioPermitido.

  // === Thresholds adaptativos (RegimeAware) ===
  regimeStrength := adxValue[0];
  // proxy: força de tendência
  // volatilityState e thresholds _eff já foram consolidados acima na mesma barra.
  // Aqui só consumimos a fonte única de verdade, sem recalcular.
  isTrend := (adxValue[0] >= MinADX_eff) and
             (ratioBW_ATR >= RatioBWATR_Min_eff) and
             (Abs(slopeN[0]) >= SlopeMin_eff);

  // regimeMercado já foi calculado de forma determinística ANTES do ORB (fonte única de verdade).
  // Proibido recalcular aqui para evitar conflito/atraso lógico.

  // [REMOVIDO – DUPLICIDADE] PG/ATRMed/volatilityState já calculados acima nesta barra.
  // Mantemos uma única fonte de verdade por barra (persistência determinística), conforme Manual.

  // ===== ORB ADAPTATIVO — fonte única de verdade =====
  // Congela a faixa ANTES do candle de rompimento.
  // Evita atrasar o VERDE/VERMELHO por incluir o candle atual no próprio limite.
  if (orComputado = 0) then
  begin
    if (minutosDesdeAbertura < Floor(ORB_Minutos)) then
    begin
      barsORB := barsORB + 1;
      orh := Max(orh, High[0]);
      orl := Min(orl, Low[0]);
    end
    else
    begin
      orComputado := 1;
      orMid := (orh + orl) * 0.5;
    end;
  end;

  // ===== VRS DINÂMICO MANHÃ × TARDE =====
  if isManha then
    begin
      tpEff := VRS_TP_Manha;
      slEff := VRS_SL_Manha;
      // Manhã: reforço mínimo de cooldown, sem checar isManha novamente
      cooldownBarsEff := Max(CooldownBars,1);
    end

  else if isTarde then
    begin
      tpEff := VRS_TP_Tarde;
      slEff := VRS_SL_Tarde;
      // tarde: efetiva com reforço de cooldown
      cooldownBarsEff := Max(CooldownBars,VRS_Cooldown_Tarde);
    end
  else
    begin
      tpEff := TP2_Mult;
      // fallback para valores atuais
      slEff := SL_Mult;
    end;

  TP1_eff := tpEff;
  TP2_eff := tpEff;
  SL_eff  := slEff;
  // Inicializa gates canônicos por candle ANTES de qualquer gatilho
  // Evita uso de allowTradeGlobal “velho” de candle anterior.
  allowTrade := True;
  allowTradeGlobal := (not travaDia) and (not travaGanho) and (cooling <= 0) and (not auditProtActive);
  gateOpen := allowTradeGlobal;

  // Janela morta opcional 12:00–13:30 (HHMM, conforme Manual NTSL)
  if VRS_JanelaMorta and (Time >= 1200) and (Time < 1330) then
  begin
    medRangeDia := atr0;  // medida canônica e já computada
    // TrueRange() é a forma válida (e coerente com o resto do script que usa TrueRange())
    allowTrade := allowTrade and (TrueRange() >= medRangeDia);
  end;

  // (removido: ORB será avaliado apenas após candleOk e cálculo de canFireORB,
  // garantindo consistência e evitando duplicidades)
  // [REMOVIDO DUPLICADO — reteste consolidado no bloco principal do ORB]
  // [REMOVIDO — cálculo duplicado do trailing; agora definido exclusivamente no bloco HasPosition (ProfitGuardian)]

  // [REMOVIDO] Cálculo alternativo de BreakEven_ATR_eff eliminado.
  // Razão científica: o ProfitGuardian dinâmico (bloco HasPosition) concentra 100% do controle
  // de breakeven e trailing (gate, prioridade e clamps), prevenindo duplicidade de estado.
  // Benefícios: elimina conflito de parâmetros, reduz risco de divergência lógica e simplifica manutenção.
  // Referências: ProfitGuardian dinâmico central (HasPosition) e clamps por tick (MinPriceIncrement).

  // [Consolidado] Removido cálculo alternativo de oracleConfidence para evitar duplicidade.
  // Mantemos apenas a versão canônica (organismo vivo) já presente no bloco oficial.
  if RiskAdapt then
    begin
      if (regimeMercado = 0) then
        begin
          TP1_eff := Max(TP1_Mult,1.8);
          TP2_eff := Max(TP2_Mult,3.0);
          SL_eff := Max(0.85,SL_Mult * 1.05);
        end
      else if (regimeMercado = 1) then
        begin
          TP1_eff := Max(0.9,TP1_Mult * 1.0);
          TP2_eff := Max(1.4,TP2_Mult * 0.9);
          SL_eff := Max(0.85,SL_Mult * 0.85);
        end
      else if (regimeMercado = 2) then
        begin
          TP1_eff := Max(1.1,TP1_Mult * 1.1);
          TP2_eff := Max(2.2,TP2_Mult * 1.3);
          SL_eff := Max(0.85,SL_Mult * 0.95);
        end
      else
        begin
          TP1_eff := Max(1.0,TP1_Mult * 0.9);
          TP2_eff := Max(1.6,TP2_Mult * 0.8);
          SL_eff := Max(0.75,SL_Mult * 0.85);
        end;
    end;

  // ================== LATERALIDADE GUARD (ANTI-CHOP, ALINHADO AO LUCRO) ==================
  // Regra: em lateralidade (regimeMercado=3), só deixa operar se houver BREAKOUT claro + candle/volume decentes.
  // Sem duplicidades: usa apenas adaptiveBlock + allowTrade (já existentes).

  adaptiveBlock := False;
  
  if (regimeMercado = 3) and (ratioBW_ATR < RatioBWATR_Min_eff) and (adx0 < MinADX_eff) and (Abs(slopeN0) < 0.05) then 
  begin
    // Snapshot causal do candle atual.
    // Evita usar bodyATR/minBodyDyn/vrsMinimo antigos antes do recálculo oficial da barra.
    bodyATR := Abs(Close[0] - Open[0]) / Max(0.0001, atr0);

    // 1) Qualidade mínima de candle usando inputs estáveis neste ponto do fluxo
    if (bodyATR < Max(0.11, MinBody_ATR * 0.90)) then
      adaptiveBlock := True;

    // 2) Volume relativo mínimo usando piso estável antes do VRS dinâmico completo
    if (vrs0 < Max(0.60, MinVRS * 0.90)) then
      adaptiveBlock := True;

    // 3) Se ainda não bloqueou: exige breakout claro OU impulso real no candle
    if (not adaptiveBlock) then
    begin
      if not( ((Close[0] > bbTop[0]) and (prevClose <= prevBbTop)) or
              ((Close[0] < bbBot[0]) and (prevClose >= prevBbBot)) or
              (HabilitarPDH_PDL and (((Close[0] > pdh) and (prevClose <= pdh)) or ((Close[0] < pdl) and (prevClose >= pdl)))) or
              ((Abs(Close[0] - Open[0]) / Max(0.0001, atr0)) >= 0.18) ) then
        adaptiveBlock := True;
    end;

    if adaptiveBlock then
      allowTrade := False;
  end;

  // ================== ORÁCULO + VRS DINÂMICO + FILTRO DE CANDLE (consolidado) ==================
  // 1) Métricas de regime/volatilidade/momentum (sempre antes de qualquer gatilho)
  // Mantemos regimeStrength normalizado; volatilityState já foi estabilizado antes (sem reescrever).
  regimeStrength := Min(100,Max(0,(adxValue[0] / 50) * 100));
  // força de tendência (ADX normalizado)
  // atrRelativo e volatilityState já definidos de modo consistente a partir de atr0/atrMed.
  // Composição de momentum NORMALIZADA [-1..1]:

  // Normalizações sem jitter: clamp determinístico e limites explícitos
  ifrNorm   := Max(-1.0, Min(1.0, (ifrValue[0] - 50.0) / 50.0));
  slopeNorm := Max(-1.0, Min(1.0, slopeN[0]));
  vrsNorm   := Max(-1.0, Min(1.0, Min(2.0, Max(0.0, vrs0)) - 1.0));
  marketMomentum := (0.50 * slopeNorm) + (0.30 * ifrNorm) + (0.20 * vrsNorm);

  // OK
  // Confiança adaptativa do oráculo (organismo vivo)
  oracleConfidence := (regimeStrength * 0.5) + (volatilityState * 15) + Min(25,Abs(marketMomentum) * 5) + Min(10,vrs0 * 5);
  oracleConfidence := Min(95,Max(15,oracleConfidence));

  // Filtro adaptativo global (anti-congestão)
  {* Histerese adaptativa + fallback seguro *}

  // Ideia: VRS mínimo e confiança passam a nascer do regime + sessão + auditor.
  // Aqui o auditor passa a ter 3 camadas:
  // 1) saúde local online (EWMA)
  // 2) walk-forward em treino
  // 3) trava soberana na fase fora da amostra
  adaptiveGateOk := False;
  // ==============================
  // WALK-FORWARD / FORA DA AMOSTRA / ANTI-OVERFITTING — Σ
  // ==============================
  wfCycleBars  := Max(2, WF_BarrasTreino + WF_BarrasOOS);
  wfFoldId     := Floor(CurrentBar() / wfCycleBars);
  wfPosInCycle := CurrentBar() - (wfFoldId * wfCycleBars);

  // Fecha o fold anterior ANTES de zerar os acumuladores.
  // Correção-Σ: antes de aprovar/rejeitar o fold, recalcula os scores com as estatísticas finais.
  // Isso evita aprovar produção usando wfPromotionScore antigo quando houve trade fechado no último candle do fold.
  if WF_Ativo and (CurrentBar() > 0) and (wfPosInCycle = 0) then
  begin
    wfTrainScore := 0.0;
    if (wfTrainTrades >= WF_MinTradesTreino) then wfTrainScore := wfTrainScore + 0.20;
    if (wfTrainWinRate >= WF_MinWinRate_Treino) then wfTrainScore := wfTrainScore + 0.20;
    if (wfTrainPF >= WF_MinPF_Treino) then wfTrainScore := wfTrainScore + 0.20;
    if (wfTrainExp_ATR >= WF_MinExp_Treino) then wfTrainScore := wfTrainScore + 0.20;
    if (wfTrainMaxDD_ATR <= WF_MaxDD_Treino) then wfTrainScore := wfTrainScore + 0.20;

    wfOOSScore := 0.0;
    if (wfOOSTrades >= WF_MinTradesOOS) then wfOOSScore := wfOOSScore + 0.25;
    if (wfOOSPF >= WF_MinPF_OOS) then wfOOSScore := wfOOSScore + 0.25;
    if (wfOOSExp_ATR >= WF_MinExp_OOS) then wfOOSScore := wfOOSScore + 0.25;
    if (wfOOSMaxDD_ATR <= WF_MaxDD_OOS) then wfOOSScore := wfOOSScore + 0.25;

    wfOverfitGap := Max(0.0, wfTrainExp_ATR - wfOOSExp_ATR);

    if (wfOOSTrades < WF_MinTradesOOS) then
      wfOverfitOk := True
    else
      wfOverfitOk := (wfOverfitGap <= WF_MaxOverfitGap);

    wfOverfitScore := 1.0 - Min(1.0, wfOverfitGap / Max(0.0001, WF_MaxOverfitGap));

    if (wfTrainExp_ATR > 0) then
      wfEdgeRatio := wfOOSExp_ATR / Max(0.0001, Abs(wfTrainExp_ATR))
    else
      wfEdgeRatio := 0.0;

    if (wfEdgeRatio < 0.0) then wfEdgeRatio := 0.0;
    if (wfEdgeRatio > 1.5) then wfEdgeRatio := 1.5;

    wfStabilityScore := (0.35 * Min(1.0, wfOOSPF / Max(0.0001, WF_MinPF_OOS))) +
                        (0.35 * Min(1.0, (wfOOSExp_ATR + 0.20) / Max(0.0001, WF_MinExp_OOS + 0.20))) +
                        (0.30 * (1.0 - Min(1.0, wfOOSMaxDD_ATR / Max(0.0001, WF_MaxDD_OOS))));

    if (wfOOSTrades < WF_MinTradesOOS) then
      wfStabilityScore := Min(wfStabilityScore, 0.45);

    wfPromotionScore := (0.40 * wfTrainScore) +
                        (0.35 * wfOOSScore) +
                        (0.15 * wfOverfitScore) +
                        (0.10 * Min(1.0, wfEdgeRatio));

    wfPromotionOk := (wfPromotionScore >= WF_MinPromotionScore) and
                     (wfStabilityScore >= WF_MinStabilityScore) and
                     wfOverfitOk;

    wfFoldApproved := (wfTrainTrades >= WF_MinTradesTreino) and
                      (wfOOSTrades   >= WF_MinTradesOOS) and
                      (wfTrainPF     >= WF_MinPF_Treino) and
                      (wfOOSPF       >= WF_MinPF_OOS) and
                      (wfTrainExp_ATR >= WF_MinExp_Treino) and
                      (wfOOSExp_ATR   >= WF_MinExp_OOS) and
                      (wfTrainMaxDD_ATR <= WF_MaxDD_Treino) and
                      (wfOOSMaxDD_ATR   <= WF_MaxDD_OOS) and
                      wfOverfitOk and
                      (wfPromotionScore >= WF_MinPromotionScore) and
                      (wfStabilityScore >= WF_MinStabilityScore);

    if wfFoldApproved then
    begin
      wfConsecApproved := wfConsecApproved + 1;
      wfConsecRejected := 0;
    end
    else
    begin
      wfConsecRejected := wfConsecRejected + 1;
      wfConsecApproved := 0;
    end;
  end;

  if (wfPosInCycle = 0) then
  begin
    wfTrainTrades     := 0;
    wfOOSTrades       := 0;
    wfTrainWins       := 0.0;
    wfOOSWins         := 0.0;
    wfTrainWinRate    := 0.0;
    wfOOSWinRate      := 0.0;
    wfTrainGain_ATR   := 0.0;
    wfTrainLoss_ATR   := 0.0;
    wfOOSGain_ATR     := 0.0;
    wfOOSLoss_ATR     := 0.0;
    wfTrainExp_ATR    := 0.0;
    wfOOSExp_ATR      := 0.0;
    wfTrainPF         := 999.0;
    wfOOSPF           := 999.0;
    wfTrainEquity_ATR := 0.0;
    wfTrainPeak_ATR   := 0.0;
    wfTrainDD_ATR     := 0.0;
    wfTrainMaxDD_ATR  := 0.0;
    wfOOSEquity_ATR   := 0.0;
    wfOOSPeak_ATR     := 0.0;
    wfOOSDD_ATR       := 0.0;
    wfOOSMaxDD_ATR    := 0.0;
    wfBlockReason     := "";
    wfPromotionScore  := 0.0;
    wfStabilityScore  := 0.0;
    wfOverfitGap      := 0.0;
    wfOverfitScore    := 1.0;
    wfPromotionOk     := False;
    wfOverfitOk       := True;
    wfOOSWarmupOk     := False;
  end
  else if (wfPosInCycle = WF_BarrasTreino) then
  begin
    // início da fase OOS: zera apenas as estatísticas OOS
    wfOOSTrades       := 0;
    wfOOSWins         := 0.0;
    wfOOSWinRate      := 0.0;
    wfOOSGain_ATR     := 0.0;
    wfOOSLoss_ATR     := 0.0;
    wfOOSExp_ATR      := 0.0;
    wfOOSPF           := 999.0;
    wfOOSEquity_ATR   := 0.0;
    wfOOSPeak_ATR     := 0.0;
    wfOOSDD_ATR       := 0.0;
    wfOOSMaxDD_ATR    := 0.0;
    wfBlockReason     := "";
  end;

  if WF_Ativo then
  begin
    wfIsTrain := (wfPosInCycle < WF_BarrasTreino);
    wfIsOOS   := not wfIsTrain;
  end
  else
  begin
    wfIsTrain := True;
    wfIsOOS   := False;
  end;

  wfGateOk  := True;
  wfPenalty := 0.0;

  // ==============================
  // TRADE AUDITOR — ESTADO DE PROTEÇÃO (primeiro, sem lag de 1 barra)
  // ==============================
  if (auditProtBarsLeft > 0) then
  begin
    auditProtBarsLeft := auditProtBarsLeft - 1;
    auditProtActive   := True;
  end
  else
    auditProtActive := False;

  // ==============================
  // AUTOAJUSTE SUPERVISIONADO SOBERANO
  // ==============================
  if isManha then
    baseVRS := VRS_Manha
  else
    baseVRS := VRS_Tarde;

  if (regimeMercado = 0) then
  begin
    regimeAdj := -0.07;
    volAdj    := -2.5;
  end
  else if (regimeMercado = 2) then
  begin
    regimeAdj := -0.02;
    volAdj    :=  0.5;
  end
  else if (regimeMercado = 3) then
  begin
    regimeAdj :=  0.08;
    volAdj    :=  4.5;
  end
  else
  begin
    regimeAdj :=  0.01;
    volAdj    :=  1.5;
  end;

  if (Time >= 1200) then
    volAdj := volAdj + 0.6;

  regimeAdjExit := 0.0;
  entropySignal := 0.0;
  minProfitATR  := 0.80 + (volatilityState * 0.16) + ((100.0 - regimeStrength) / 140.0);

  if (auditTrades >= (4 + volatilityState)) then
  begin
    if (auditExpectancy_ATR < 0) then
    begin
      regimeAdjExit := regimeAdjExit + Min(0.10, Abs(auditExpectancy_ATR) * 0.07);
      entropySignal := entropySignal + Min(5.0, Abs(auditExpectancy_ATR) * 2.20);
    end
    else
    begin
      regimeAdjExit := regimeAdjExit - Min(0.05, auditExpectancy_ATR * 0.05);
      entropySignal := entropySignal - Min(1.50, auditExpectancy_ATR * 1.20);
    end;

    if (auditProfitFactor < 1.0) then
    begin
      regimeAdjExit := regimeAdjExit + Min(0.08, (1.0 - auditProfitFactor) * 0.12);
      entropySignal := entropySignal + Min(3.5, (1.0 - auditProfitFactor) * 5.00);
    end
    else
    begin
      regimeAdjExit := regimeAdjExit - Min(0.04, (auditProfitFactor - 1.0) * 0.04);
      entropySignal := entropySignal - Min(1.20, (auditProfitFactor - 1.0) * 1.80);
    end;

    if (auditEwmaWin < 0.52) then
    begin
      regimeAdjExit := regimeAdjExit + Min(0.08, (0.52 - auditEwmaWin) * 0.18);
      entropySignal := entropySignal + Min(3.0, (0.52 - auditEwmaWin) * 8.00);
    end
    else
    begin
      regimeAdjExit := regimeAdjExit - Min(0.03, (auditEwmaWin - 0.52) * 0.08);
      entropySignal := entropySignal - Min(1.00, (auditEwmaWin - 0.52) * 3.50);
    end;

    if (auditMaxDD_ATR > minProfitATR) then
    begin
      regimeAdjExit := regimeAdjExit + Min(0.10, (auditMaxDD_ATR - minProfitATR) * 0.05);
      entropySignal := entropySignal + Min(4.0, (auditMaxDD_ATR - minProfitATR) * 1.10);
    end;
  end;

  if auditProtActive then
  begin
    regimeAdjExit := regimeAdjExit + 0.10;
    entropySignal := entropySignal + 4.0;
  end;

  // ==============================
  // SCORE FORMAL WF/OOS — CONTROLE DE OVERFITTING
  // ==============================
  wfTrainScore := 0.0;
  if (wfTrainTrades >= WF_MinTradesTreino) then wfTrainScore := wfTrainScore + 0.20;
  if (wfTrainWinRate >= WF_MinWinRate_Treino) then wfTrainScore := wfTrainScore + 0.20;
  if (wfTrainPF >= WF_MinPF_Treino) then wfTrainScore := wfTrainScore + 0.20;
  if (wfTrainExp_ATR >= WF_MinExp_Treino) then wfTrainScore := wfTrainScore + 0.20;
  if (wfTrainMaxDD_ATR <= WF_MaxDD_Treino) then wfTrainScore := wfTrainScore + 0.20;

  wfOOSWarmupOk := (wfOOSTrades > 0) and (wfOOSTrades < WF_MinTradesOOS);

  wfOOSScore := 0.0;
  if (wfOOSTrades >= WF_MinTradesOOS) then wfOOSScore := wfOOSScore + 0.25;
  if (wfOOSPF >= WF_MinPF_OOS) then wfOOSScore := wfOOSScore + 0.25;
  if (wfOOSExp_ATR >= WF_MinExp_OOS) then wfOOSScore := wfOOSScore + 0.25;
  if (wfOOSMaxDD_ATR <= WF_MaxDD_OOS) then wfOOSScore := wfOOSScore + 0.25;

  if wfOOSWarmupOk then
    wfOOSScore := Max(wfOOSScore, 0.25);

  wfOverfitGap := Max(0.0, wfTrainExp_ATR - wfOOSExp_ATR);

  if (wfOOSTrades < WF_MinTradesOOS) then
    wfOverfitOk := True
  else
    wfOverfitOk := (wfOverfitGap <= WF_MaxOverfitGap);

  wfOverfitScore := 1.0 - Min(1.0, wfOverfitGap / Max(0.0001, WF_MaxOverfitGap));

  if (wfTrainExp_ATR > 0) then
    wfEdgeRatio := wfOOSExp_ATR / Max(0.0001, Abs(wfTrainExp_ATR))
  else
    wfEdgeRatio := 0.0;

  if (wfEdgeRatio < 0.0) then wfEdgeRatio := 0.0;
  if (wfEdgeRatio > 1.5) then wfEdgeRatio := 1.5;

  wfStabilityScore := (0.35 * Min(1.0, wfOOSPF / Max(0.0001, WF_MinPF_OOS))) +
                      (0.35 * Min(1.0, (wfOOSExp_ATR + 0.20) / Max(0.0001, WF_MinExp_OOS + 0.20))) +
                      (0.30 * (1.0 - Min(1.0, wfOOSMaxDD_ATR / Max(0.0001, WF_MaxDD_OOS))));

  if (wfOOSTrades < WF_MinTradesOOS) then
    wfStabilityScore := Min(wfStabilityScore, 0.45);

  wfPromotionScore := (0.40 * wfTrainScore) +
                      (0.35 * wfOOSScore) +
                      (0.15 * wfOverfitScore) +
                      (0.10 * Min(1.0, wfEdgeRatio));

  wfPromotionOk := (wfPromotionScore >= WF_MinPromotionScore) and
                   (wfStabilityScore >= WF_MinStabilityScore) and
                   wfOverfitOk;

  // ==============================
  // SUPERVISÃO WF/OOS SOBERANA
  // ==============================
  if WF_Ativo then
  begin
    if wfIsOOS then
    begin
      if (wfTrainTrades < WF_MinTradesTreino) then
      begin
        wfGateOk := False;
        wfBlockReason := "WF=TREINO_INSUF";
      end
      else if (wfTrainWinRate < WF_MinWinRate_Treino) then
      begin
        wfGateOk := False;
        wfBlockReason := "WF=WINRATE_TREINO_FRACA";
      end
      else if (wfTrainPF < WF_MinPF_Treino) then
      begin
        wfGateOk := False;
        wfBlockReason := "WF=PF_TREINO_FRACO";
      end
      else if (wfTrainExp_ATR < WF_MinExp_Treino) then
      begin
        wfGateOk := False;
        wfBlockReason := "WF=EXP_TREINO_FRACA";
      end
      else if (wfTrainMaxDD_ATR > WF_MaxDD_Treino) then
      begin
        wfGateOk := False;
        wfBlockReason := "WF=DD_TREINO_ALTO";
      end;

      if wfGateOk then
      begin
        if wfOOSWarmupOk then
        begin
          wfPenalty := wfPenalty + Min(WF_PenalidadeOOS * 0.50, (WF_MinTradesOOS - wfOOSTrades) * 0.50);
          if (wfBlockReason = "") then
            wfBlockReason := "WF=OOS_AQUECENDO";
        end;

        if (wfOOSTrades >= WF_MinTradesOOS) then
        begin
          if (wfOOSPF < WF_MinPF_OOS) then
          begin
            wfPenalty := wfPenalty + Min(WF_PenalidadeOOS, (WF_MinPF_OOS - wfOOSPF) * 12.0);
            if (wfBlockReason = "") then
              wfBlockReason := "WF=PF_OOS_FRACO";
          end;

          if (wfOOSExp_ATR < WF_MinExp_OOS) then
          begin
            wfPenalty := wfPenalty + Min(WF_PenalidadeOOS, (WF_MinExp_OOS - wfOOSExp_ATR) * 30.0);
            if (wfBlockReason = "") then
              wfBlockReason := "WF=EXP_OOS_FRACA";
          end;

          if (wfOOSMaxDD_ATR > WF_MaxDD_OOS) then
          begin
            wfPenalty := wfPenalty + Min(WF_PenalidadeOOS, (wfOOSMaxDD_ATR - WF_MaxDD_OOS) * 3.0);
            if (wfBlockReason = "") then
              wfBlockReason := "WF=DD_OOS_ALTO";
          end;

          if WF_RequerOOSPositivo and (wfOOSExp_ATR < 0) then
          begin
            wfPenalty := wfPenalty + Min(WF_PenalidadeOOS, Abs(wfOOSExp_ATR) * 22.0);
            if (wfBlockReason = "") then
              wfBlockReason := "WF=OOS_NEGATIVO";
          end;

          if not wfOverfitOk then
          begin
            wfPenalty := wfPenalty + WF_PenalidadeOverfit;
            if (wfBlockReason = "") then
              wfBlockReason := "WF=OVERFIT_TREINO_MAIOR_OOS";
          end;

          if (wfPromotionScore < WF_MinPromotionScore) then
          begin
            wfPenalty := wfPenalty + Min(WF_PenalidadeOOS, (WF_MinPromotionScore - wfPromotionScore) * 10.0);
            if (wfBlockReason = "") then
              wfBlockReason := "WF=SCORE_PROMOCAO_FRACO";
          end;

          if (wfStabilityScore < WF_MinStabilityScore) then
          begin
            wfPenalty := wfPenalty + Min(WF_PenalidadeOOS, (WF_MinStabilityScore - wfStabilityScore) * 8.0);
            if (wfBlockReason = "") then
              wfBlockReason := "WF=ESTABILIDADE_FRACA";
          end;
        end;
      end;

      if (not wfGateOk) or (wfPenalty >= WF_PenalidadeOOS) then
      begin
        wfGateOk := False;
        auditProtBarsLeft := Max(auditProtBarsLeft, WF_BloqueioBarras);

        if (auditProtReason = "") then
          auditProtReason := "WF_BLOCK";
      end;
    end;
  end;

  // ==============================
  // CRITÉRIO FORMAL DE PROMOÇÃO PARA PRODUÇÃO
  // Unifica governança WF + Produção e elimina parâmetro fantasma.
  // ==============================
  prodReady := (not WF_Ativo) or
               (
                 (wfConsecApproved >= Max(WF_MinFoldsApproved, Prod_RequerFoldsAprovados)) and
                 (wfConsecRejected < Min(WF_MaxFoldsRejected, Prod_BloqueiaSeRejeicoes)) and
                 (wfPromotionScore >= Prod_MinPromotionScore) and
                 wfPromotionOk
               );

  prodBlock := False;
  prodReason := "";

  if Prod_BloquearExecucaoReal and (not prodReady) then
  begin
    prodBlock := True;

    if (wfConsecApproved < Max(WF_MinFoldsApproved, Prod_RequerFoldsAprovados)) then
      prodReason := "PROD=FOLDS_INSUF"
    else if (wfConsecRejected >= Min(WF_MaxFoldsRejected, Prod_BloqueiaSeRejeicoes)) then
      prodReason := "PROD=REJEICOES"
    else if (wfPromotionScore < Prod_MinPromotionScore) then
      prodReason := "PROD=SCORE_FRACO"
    else
      prodReason := "PROD=WF_NAO_PROMOVIDO";
  end;
  
  if prodBlock and (not SomenteSinais) then
  begin
    allowTradeGlobal := False;
    adaptiveGateOk   := False;
    gateOpen         := False;

    if (xaiBlockReason = "") then
      xaiBlockReason := prodReason;
  end;

  if (wfPenalty > 0) then
  begin
    regimeAdjExit := regimeAdjExit + Min(0.18, wfPenalty * 0.012);
    entropySignal := entropySignal + Min(8.0, wfPenalty);
  end;

  // tendência muito forte e momentum limpo aliviam; mercado morto aperta
  if (regimeStrength >= (MinADX_eff + 8.0)) then
  begin
    regimeAdj := regimeAdj - 0.03;
    volAdj    := volAdj - 1.0;
  end;

  if (Abs(marketMomentum) > 0.45) then
  begin
    regimeAdj := regimeAdj - 0.02;
    volAdj    := volAdj - 0.8;
  end
  else if (Abs(marketMomentum) < 0.12) then
  begin
    regimeAdj := regimeAdj + 0.02;
    volAdj    := volAdj + 0.8;
  end;

  // ==========================================================
  // CUSTO REAL EFETIVO + PISO LÍQUIDO MÍNIMO + SIZING DINÂMICO — Σ
  // ==========================================================
  if Exec_UsarCustosReais then
    execCostTicksEff := Exec_SlippageIn_Ticks
                      + Exec_SlippageOut_Ticks
                      + Exec_Spread_Ticks
                      + Exec_Delay_Ticks
                      + Exec_Corretagem_Ticks
                      + Exec_Emolumentos_Ticks
  else
    execCostTicksEff := 0.0;

  // Penalização dinâmica: custo real cresce quando o mercado fica hostil.
  if (volatilityState = 2) then
    execCostTicksEff := execCostTicksEff + Exec_PenalidadeVolAlta_Ticks;

  if WF_Ativo and wfIsOOS then
    execCostTicksEff := execCostTicksEff + Exec_PenalidadeOOS_Ticks;

  if (regimeMercado = 1) then
    execCostTicksEff := execCostTicksEff + Max(0.0, Exec_Spread_Ticks * 0.50);

  if (Abs(marketMomentum) < 0.12) then
    execCostTicksEff := execCostTicksEff + Max(0.0, Exec_Delay_Ticks * 0.50);

  // Gap/evento extremo aumenta fricção e impede ilusão de proteção.
  extremeGapActive := False;
  gapRiskATR := 0.0;

  if CurrentBar() > 0 then
    gapRiskATR := Abs(Open[0] - Close[1]) / Max(0.0001, atr0);

  if gapRiskATR >= Exec_GapEmergency_ATR then
  begin
    extremeGapActive := True;
    execCostTicksEff := execCostTicksEff + Exec_GapEmergency_Ticks;
  end;

  costFloorPx  := execCostTicksEff * MinPriceIncrement;
  costFloorATR := Max(Exec_MinNet_ATR + (volatilityState * 0.02), costFloorPx / Max(0.0001, atr0));

  if WF_Ativo and wfIsOOS and (not wfPromotionOk) then
    costFloorATR := costFloorATR + ((Exec_PenalidadeOOS_Ticks * MinPriceIncrement) / Max(0.0001, atr0));

  // Piso líquido mínimo soberano: se custo/fricção sobe, a saída/entrada exige edge maior.
  minProfitATR := Max(minProfitATR, costFloorATR + 0.05 + (volatilityState * 0.03));

  execNetEdgeATR := Max(0.0, minProfitATR - costFloorATR);

  // ================= POSITION SIZING DINÂMICO POR REGIME/RISCO =================
  qtyEntryEff := QtdOrdem;

  if SizingDinamico then
  begin
    // Ambiente favorável: tendência limpa + auditor/WF saudáveis + fricção controlada.
    if (regimeMercado = 2) and
       (volatilityState <= 1) and
       (regimeStrength >= (MinADX_eff + 4.0)) and
       (auditExpectancy_ATR >= 0) and
       wfGateOk and
       ((not WF_Ativo) or wfPromotionOk or wfIsTrain) and
       (costFloorATR <= Exec_MinNet_ATR * 1.35) then
      qtyEntryEff := qtyEntryEff + Sizing_RegimeTrendBonus;

    // Lateralidade, vol alta, gap e custo alto reduzem exposição.
    if (regimeMercado = 1) then
      qtyEntryEff := qtyEntryEff - 1;

    if (volatilityState = 2) then
      qtyEntryEff := qtyEntryEff - Sizing_VolAltaRedutor;

    if extremeGapActive then
      qtyEntryEff := qtyEntryEff - 1;

    if (costFloorATR > Exec_MinNet_ATR * 1.50) then
      qtyEntryEff := qtyEntryEff - 1;

    // Auditor ruim reduz; captura ruim do AMARELO também reduz.
    if (auditExpectancy_ATR < 0) then
      qtyEntryEff := qtyEntryEff - 1;

    if (auditMaxDD_ATR > Sizing_AuditPain) then
      qtyEntryEff := qtyEntryEff - 1;

    if (auditTrades >= 2) and (not yellowCaptureOk) then
      qtyEntryEff := qtyEntryEff - 1;

    // Walk-forward pressionado reduz agressão.
    if (wfPenalty >= (WF_PenalidadeOOS * Sizing_WFPenalty)) then
      qtyEntryEff := qtyEntryEff - 1;

    if WF_Ativo and wfIsOOS and (not wfPromotionOk) then
      qtyEntryEff := qtyEntryEff - 1;

    // Auditor bom e ambiente favorável permitem reforço somente com custo controlado.
    if (auditExpectancy_ATR > Sizing_AuditGain) and
       (auditProfitFactor > 1.20) and
       (wfPenalty <= 1.0) and
       (volatilityState <= 1) and
       (regimeMercado = 2) and
       (costFloorATR <= Exec_MinNet_ATR * 1.25) and
       yellowCaptureOk and
       ((not WF_Ativo) or wfPromotionOk or wfIsTrain) then
      qtyEntryEff := qtyEntryEff + 1;
  end;

  if (not wfGateOk) or auditProtActive or travaDia or travaGanho or (prodBlock and (not SomenteSinais)) then
    qtyEntryEff := QtdMin;

  if (qtyEntryEff < QtdMin) then
    qtyEntryEff := QtdMin;

  if (qtyEntryEff > QtdMax) then
    qtyEntryEff := QtdMax;

  if (Abs(marketMomentum) < 0.12) then
  begin
    regimeAdj := regimeAdj + 0.02;
    volAdj    := volAdj + 0.8;
  end;

  // ==============================
  // VRS mínimo dinâmico — supervisionado por sessão/regime/auditor/WF
  // ==============================
  if VRS_Dinamico then
  begin
    vrsMinimo := baseVRS + regimeAdj + (regimeAdjExit * 0.65);

    if (regimeStrength >= (MinADX_eff + 6.0)) then
      vrsMinimo := vrsMinimo - 0.03;

    if (Abs(marketMomentum) > 0.45) then
      vrsMinimo := vrsMinimo - 0.02
    else if (Abs(marketMomentum) < 0.12) then
      vrsMinimo := vrsMinimo + 0.02;

    candidateFloor := (MinVRS * (0.78 + (volatilityState * 0.04))) + Max(0.0, regimeAdjExit * 0.18);
    candidateCeil  := (baseVRS + 0.10) - Min(0.10, regimeStrength / 300.0) + Max(0.0, regimeAdjExit * 0.25);

    if (auditExpectancy_ATR > 0) then
      candidateCeil := candidateCeil - Min(0.05, auditExpectancy_ATR * 0.05)
    else
      candidateFloor := candidateFloor + Min(0.05, Abs(auditExpectancy_ATR) * 0.04);

    if auditProtActive then
      candidateFloor := candidateFloor + 0.03;

    if (wfPenalty > 0) then
      candidateFloor := candidateFloor + Min(0.06, wfPenalty * 0.006);

    if (candidateCeil < candidateFloor) then
      candidateCeil := candidateFloor;

    vrsMinimo := Max(candidateFloor, Min(candidateCeil, vrsMinimo));
  end
  else
  begin
    candidateFloor := MinVRS * (0.78 + (volatilityState * 0.04));
    candidateCeil  := candidateFloor + 0.10 + Max(0.0, regimeAdjExit * 0.20);

    if auditProtActive then
      candidateFloor := candidateFloor + 0.03;

    if (wfPenalty > 0) then
      candidateFloor := candidateFloor + Min(0.06, wfPenalty * 0.006);

    if (candidateCeil < candidateFloor) then
      candidateCeil := candidateFloor;

    vrsMinimo := Max(candidateFloor, Min(candidateCeil, MinVRS + (regimeAdjExit * 0.20)));
  end;

  vrsReq := Max(baseVRS + regimeAdj + (regimeAdjExit * 0.65), vrsMinimo);

  candidateFloor := Max(
                      MinVRS * (0.80 + (volatilityState * 0.03)),
                      (baseVRS - 0.08) + (volatilityState * 0.02) + Max(0.0, regimeAdjExit * 0.12)
                    );

  candidateCeil  := (baseVRS + 0.10) - Min(0.10, regimeStrength / 320.0) + Max(0.0, regimeAdjExit * 0.20);

  if (regimeMercado = 3) then
    candidateFloor := candidateFloor + 0.02;

  if auditProtActive then
    candidateFloor := candidateFloor + 0.03;

  if (wfPenalty > 0) then
    candidateFloor := candidateFloor + Min(0.08, wfPenalty * 0.008);

  if (candidateCeil < candidateFloor) then
    candidateCeil := candidateFloor;

  vrsReq := Max(candidateFloor, Min(candidateCeil, vrsReq));

  // ==============================
  // Confiança mínima supervisionada — dinâmica + WF/OOS
  // ==============================
  candidateFloor := 22.0 + (volatilityState * 2.5) + Max(0.0, entropySignal * 0.35) + Max(0.0, regimeAdjExit * 20.0);
  candidateCeil  := 60.0 - Min(11.0, regimeStrength / 8.5);

  if (regimeMercado = 3) then
    candidateCeil := candidateCeil + 4.0;

  if (auditExpectancy_ATR > 0) then
    candidateCeil := Max(38.0, candidateCeil - Min(5.0, auditExpectancy_ATR * 3.0))
  else
    candidateFloor := candidateFloor + Min(6.0, Abs(auditExpectancy_ATR) * 3.0);

  if auditProtActive then
    candidateFloor := candidateFloor + 4.0;

  if (wfPenalty > 0) then
    candidateFloor := candidateFloor + Min(10.0, wfPenalty);

  if (Abs(marketMomentum) > 0.45) then
    candidateCeil := candidateCeil - 1.0
  else if (Abs(marketMomentum) < 0.12) then
    candidateFloor := candidateFloor + 1.0;

  if (candidateCeil < candidateFloor) then
    candidateCeil := candidateFloor;

  confReq := 27.0
             + (volatilityState * 2.5)
             + volAdj
             + ((100.0 - regimeStrength) / 22.0)
             + entropySignal
             + Max(0.0, regimeAdjExit * 18.0);

  confReq := Max(candidateFloor, Min(candidateCeil, confReq));

  gateAuditorOk := (not auditProtActive) and wfGateOk;

  // Gate adaptativo soberano
  adaptiveGateOk := True;

  if auditProtActive then
    adaptiveGateOk := False
  else if (not gateAuditorOk) then
  begin
    adaptiveGateOk := False;
    if (wfBlockReason <> "") then
      xaiBlockReason := wfBlockReason;
  end
  else if (regimeStrength < MinADX_eff) and
          (vrs0 < vrsReq) and
          (oracleConfidence < confReq) and
          ((Abs(marketMomentum) < 0.18) or (regimeMercado = 3)) then
    adaptiveGateOk := False;

//==================================
  if ModoDebug then
    PlotText("VRS|CHECK",clWhite,1,6,Close[0] + 0.60 * atr0);

  // Filtro de Candle oficial + medidas objetivas de qualidade
  body := Abs(Close - Open);
  upperWick := High - Max(Open,Close);
  lowerWick := Min(Open,Close) - Low;
  if (atr0 > 0) then
    bodyATR := body / atr0
  else
    bodyATR := 0;
  if (body > 0) then
    wickRatio := (upperWick + lowerWick) / body
  else
    wickRatio := 999;

  // Doji oficial do Manual: C_Doji(percentual) retorna Integer (0/1).
  // Convertemos para Boolean determinístico e SEM função inventada.
  doji := BloquearDoji and (C_Doji(DojiPercent) = 1);

  // Thresholds dinâmicos autorregulados por estado de volatilidade (0=baixa,1=média,2=alta)
  // Corpo: ±0.02 ATR | Pavio: ±0.05 — atende pedido e mantém BloquearDoji ativo
  if (volatilityState = 0) then
    begin
      minBodyDyn := Max(0.06, MinBody_ATR + 0.02);
      wickMaxDyn := Max(0.70, WickRatioMax - 0.05);
    end
  else if (volatilityState = 2) then
    begin
      minBodyDyn := Max(0.06, MinBody_ATR - 0.02);
      wickMaxDyn := Max(0.70, WickRatioMax + 0.05);
    end
  else
    begin
      minBodyDyn := Max(0.06, MinBody_ATR);
      wickMaxDyn := Max(0.70, WickRatioMax);
    end;

  // qualidade do candle isolada do gate de volume; VRS é checado nos gatilhos (opBuySeed/opSellSeed)
  candleOk := (not doji)
              and (bodyATR >= (minBodyDyn * 0.95))
              and (wickRatio <= (wickMaxDyn * 1.05));

  // DEBUG opcional por barra (exibe primeira causa de bloqueio)
  if ModoDebug and (not candleOk) then
  begin
    if doji then
      PlotText("BLOCK:DOJI",  clSilver, 1, 6, Low[0] - 0.20 * atr0)
    else if bodyATR < (minBodyDyn * 0.95) then
      PlotText("BLOCK:BODY",  clSilver, 1, 6, Low[0] - 0.20 * atr0)
    else if wickRatio > (wickMaxDyn * 1.05) then
      PlotText("BLOCK:WICK",  clSilver, 1, 6, Low[0] - 0.20 * atr0)
    
    else if vrs0 < vrsMinimo then PlotText("BLOCK:VRS", clSilver, 1, 6, Low[0] - 0.20 * atr0);
  end;

  // === ORB gate dinâmico priorizando janela, // VRS e "gate" adaptativo global
  // Usa o horário consolidado e o gate adaptativo já calculado no organismo (adaptiveGateOk)
  canFireORB := HabilitarORB
                and horarioPermitido
                and adaptiveGateOk
                and (((not ORB_RequerVRS) or (vrs0 >= vrsMinimo)))
                and candleOk;

  // (Removido para evitar duplicidade: mantém-se o filtro ÚNICO já definido acima
  // com C_Doji() do Manual + thresholds dinâmicos por ATR/pavio/VRS).
  // Manual C_Doji — ver documentação oficial de C_Doji() na seção de Candlestick
  // ================== DETECÇÃO DE SINAIS (pintura destravada) ==================
  // 1) Rompimentos de banda com CRUZAMENTO confirmado (sem deslocar sinais)
  //    Regras: o fechamento anterior precisa estar do "outro lado" da banda.
  rompAlta1 := (prevClose <= prevBbTop) and (Close[0] > bbTop[0]);
  rompBaixa1 := (prevClose >= prevBbBot) and (Close[0] < bbBot[0]);

   // Gate de volume/candle e IFR anti-falso
  rompAlta1Vol := rompAlta1 and (vrs0 >= vrsMinimo) and candleOk;
  rompBaixa1Vol := rompBaixa1 and (vrs0 >= vrsMinimo) and candleOk;
  falsoAlta := UsarIFR_Extremos and (ifrValue[0] > IFR_FalsoRomp_Compra);
  falsoBaixa := UsarIFR_Extremos and (ifrValue[0] < IFR_FalsoRomp_Venda);
  opBuySeed := rompAlta1Vol and ( not falsoAlta);
  opSellSeed := rompBaixa1Vol and ( not falsoBaixa);

  // ===== CONFIRMAÇÃO 2 PASSOS =====
  // Regras:
  // - Confirm2Steps = True  -> exige seed + confirmação.
  // - ConfirmaCloseBB = True -> confirmação exclusivamente no fechamento.
  // - Confirm2Steps = False -> modo de 1 passo, usando apenas o seed validado.
  if Confirm2Steps then
  begin
    if ConfirmaCloseBB then
    begin
      buyCont  := opBuySeed  and ((Close[safeOffset] <= bbMid[safeOffset]) and (Close[0] > bbMid[0]));
      sellCont := opSellSeed and ((Close[safeOffset] >= bbMid[safeOffset]) and (Close[0] < bbMid[0]));
    end
    else
    begin
      buyCont  := opBuySeed  and ((High[0] > bbMid[0])
                   or ((Close[safeOffset] <= bbMid[safeOffset]) and (Close[0] > bbMid[0])));
      sellCont := opSellSeed and ((Low[0] < bbMid[0])
                   or ((Close[safeOffset] >= bbMid[safeOffset]) and (Close[0] < bbMid[0])));
    end;
  end
  else
  begin
    buyCont  := opBuySeed;
    sellCont := opSellSeed;
  end;

  // Acelerador ADX com histerese leve e coerência direcional (reduz serrilhado intrabar)
  if HabilitarAcelerador
     and ((adx0 - adxPrev) > Acelerador_ADX_MinDelta)
     and AcelIgnoraTrend
     and candleOk
     and horarioPermitido
     and allowTrade
     and allowTradeGlobal
     and adaptiveGateOk
     and (vrs0 >= vrsMinimo) then
  begin
    if (marketMomentum >= 0.05) and (close0 > bbMid0) and (not sellCont) then
      buyFast := True;

    if (marketMomentum <= -0.05) and (close0 < bbMid0) and (not buyCont) then
      sellFast := True;
  end;

   // ===== MICRO-PODAS (apenas refinamento local de confirmação; resto das podas permanece unificado) =====
  if RegimeAware and (regimeStrength >= (MinADX_eff + 6)) and (volatilityState = 2) then
    begin
      // Em tendência muito forte + vol. alta, exigimos convicção: afastamento mínimo de 0.10*ATR da mid
      if buyCont then
        buyCont := (close0 >= bbMid0 + 0.10 * atr0);
      if sellCont then
        sellCont := (close0 <= bbMid0 - 0.10 * atr0);
    end;

  // 4) PDH/PDL (do DIA ANTERIOR) — PINTURA com PODA por consistência
  // Regra: PDH/PDL só tem autoridade quando há "pressão real" (VRS) + candleOk.
  // Isso reduz falso rompimento e evita mais um “motor” disparando sem contexto.
  if HabilitarPDH_PDL and (pdh > 0) and (pdl > 0) and candleOk and (vrs0 >= (vrsMinimo + 0.05)) then
  begin
    if (Close[0] > pdh) and (prevClose <= pdh) then
      buyFast := True;

    if (Close[0] < pdl) and (prevClose >= pdl) then
      sellFast := True;
  end;

  // 5/6/7) GOVERNADOR CIENTÍFICO DE COMPLEXIDADE (ANTI-OVERENGINEERING)
  // Regra de ouro: POR CANDLE, apenas 1 motor de continuação pode ter autoridade.
  // Prioridade objetiva (lucro > “inteligência”):
  //   (1) GapFade (somente janela curta e somente se gap for relevante)
  //   (2) TrendMotor = BBMidBounce (se tendência)
  //   (3) RangeMotor = NR/Inside (se não tendência)
  //
  // Resultado: remove “paralisia decisória” e conflitos de motores sem criar funções, sem mexer no FSM e sem quebrar ORB.
  // (Determinismo preservado: tudo por snapshots e condições diretas.)

  // --- (1) GAPFADE TEM PRIORIDADE ABSOLUTA NA JANELA EM QUE FAZ SENTIDO ---
  if HabilitarGapFade and candleOk and (minutosDesdeAbertura <= 30) and (Abs(percentGap) >= GapFade_MinPercent) then
  begin
    // poda dura: quando GapFade está ativo, nenhum outro motor concorre neste candle
    buyFast  := False;
    sellFast := False;

    // executa GapFade nos dois lados: gap positivo e gap negativo
    if (gapAbertura > 0) and (High[0] < prevHigh) then
      sellCont := True;

    if (gapAbertura < 0) and (Low[0] > prevLow) then
      buyCont := True;
  end

  else
  begin
    // --- (2) TREND MOTOR: BBMidBounce (somente quando há tendência) ---
    if isTrend then
    begin
      // poda: em tendência, zera motor de range (NR) por design (1 motor por candle)
      buyCont  := False;
      sellCont := False;

      if HabilitarBBMidBounce and candleOk then
      begin
        if (Close > bbMid) and (Low <= bbMid) then
          buyFast := True;

        if (Close < bbMid) and (High >= bbMid) then
          sellFast := True;
      end;
    end
    else
    begin
      // --- (3) RANGE MOTOR: NR/Inside Break (somente quando NÃO há tendência) ---
      // poda: em range, zera motor de tendência por design (1 motor por candle)
      buyFast  := False;
      sellFast := False;

      trNow := TrueRange();
      trAvg := AvgTrueRange(NR_Barras,2);

      if HabilitarNR and (CurrentBar() >= NR_Barras) then
      begin
        // Faixa NR deve ser a janela FECHADA anterior, não incluindo o candle de rompimento.
        nrTop := High[1];
        nrBot := Low[1];

        for lagIdx := 2 to NR_Barras do
        begin
          nrTop := Max(nrTop, High[lagIdx]);
          nrBot := Min(nrBot, Low[lagIdx]);
        end;

        isNR  := (trAvg > 0) and (trNow <= (NR_Fator * trAvg));
      end
      else
      begin
        isNR  := False;
        nrTop := High[0];
        nrBot := Low[0];
      end;

      if HabilitarNR and isNR and candleOk and (vrs0 >= vrsMinimo) then
      begin
        if (Close[0] > nrTop) then
          buyCont := True;

        if (Close[0] < nrBot) then
          sellCont := True;
      end;
    end;
  end;

  // (Removido: ORB é construído somente no bloco adaptativo anterior,
  // evitando faixa duplicada e mantendo coerência com o fechamento fora da ORB).

  // 8) ORB Break — PINTURA com gate unificado (canFireORB)
  // ORB_RequerClose agora é autoridade real:
  // - True  -> exige rompimento no fechamento
  // - False -> aceita rompimento intrabar
  if (orComputado = 1) and canFireORB and (Time >= tempoLimiteORB) then
  begin
    if ORB_RequerClose then
    begin
      if (Close[0] > orh) and ((Close[0] - orh) >= (0.10 * atr0)) then
        buyFast := True
      else if ORB_PermiteReteste and candleOk and (High >= orh)
              and (Abs(Close[0] - orh) <= (orbStopRetesteATR_eff * atr0)) then
        buyFast := True;

      if (Close[0] < orl) and ((orl - Close[0]) >= (0.10 * atr0)) then
        sellFast := True
      else if ORB_PermiteReteste and candleOk and (Low <= orl)
              and (Abs(Close[0] - orl) <= (orbStopRetesteATR_eff * atr0)) then
        sellFast := True;
    end
    else
    begin
      if (High > orh) and ((High - orh) >= (0.10 * atr0)) then
        buyFast := True
      else if ORB_PermiteReteste and candleOk and (High >= orh)
              and (Abs(Close[0] - orh) <= (orbStopRetesteATR_eff * atr0)) then
        buyFast := True;

      if (Low < orl) and ((orl - Low) >= (0.10 * atr0)) then
        sellFast := True
      else if ORB_PermiteReteste and candleOk and (Low <= orl)
              and (Abs(Close[0] - orl) <= (orbStopRetesteATR_eff * atr0)) then
        sellFast := True;
    end;
  end;

  // ===== Filtro de Gap de Abertura (janela dinâmica por regime) =====
  gapBlock := False;
  if FiltroGapAbertura and (Abs(percentGap) >= MaxGapPercent) then
    begin
      // Janela = 15..40m conforme força de tendência/volatilidade (mais fraco => mais longa)
      janelaMin := 15 + Round(Max(0,25 - Round(regimeStrength / 4)));
      tempoEstab := MinutesToTime(TimeToMinutes(horaInicioReal) + janelaMin);
      gapBlock := (Time < tempoEstab);
    end;
  // OK

if gapBlock then
    begin
      // Nos minutos iniciais com gap grande: prioriza apenas GapFade nos dois sentidos
      if not (HabilitarGapFade and (Abs(percentGap) >= GapFade_MinPercent)) then
      begin
        buyCont  := False;
        sellCont := False;
        buyFast  := False;
        sellFast := False;

        if ModoDebug then
        begin
          // Mapeia estado → rótulo textual (evita conversão numérica e concatenação inválida)
          if      (paintStateSeries[0] = 2)  then stateLabel := "STATE=AMARELO"
          else if (paintStateSeries[0] = 1)  then stateLabel := "STATE=VERDE"
          else if (paintStateSeries[0] = -1) then stateLabel := "STATE=VERMELHO"
          else                                   stateLabel := "STATE=NEUTRO";

          PlotText(stateLabel, clSilver, 1, 6, Low[0] - 0.20 * atr0);
        end;
      end;
    end;

  // ==============================
  // TRADE AUDITOR — XRAY (tempo real, Manual: Função XRay)
  // - Mostra métricas no botão Raio-X do Gerenciador de Automações (quando disponível).
  // - Não interfere no motor de cores; é observabilidade pura.
  // ==============================
  if UsarXRay then
  begin
    XRay("AUDIT_SCORE", True, "AUDITOR", auditScore, clLime, clSilver);
    XRay("AUDIT_PNL_ATR", True, "AUDITOR", auditPnL_ATR, clLime, clSilver);
    XRay("EW_WIN(%)", True, "AUDITOR", (auditEwmaWin * 100), clLime, clSilver);
    XRay("PF", True, "AUDITOR", auditProfitFactor, clLime, clSilver);
    XRay("EXP_ATR", True, "AUDITOR", auditExpectancy_ATR, clLime, clSilver);
    XRay("MAXDD_ATR", True, "AUDITOR", auditMaxDD_ATR, clRed, clSilver);
    XRay("PROTECT_BARS", auditProtActive, "AUDITOR", auditProtBarsLeft, clRed, clSilver);
    XRay("REV_PROFIT_ATR", True, "EXIT", currentProfitATR, clYellow, clSilver);
  end;

  // ===== ORÁCULO QUÂNTICO - DETECÇÃO DO PONTO ÓTIMO CIENTÍFICO ======================
  // SISTEMA BASEADO EM FÍSICA ESTATÍSTICA E TEORIA DA DECISÃO BAYESIANA
  // (Manual NTSL 5.1-5.4 - Controle de Fluxo; 16.1-16.8 - Candlestick)

  // Sinais base após podas globais — exclusão mútua determinística
  buyOn  := (buyFast  or buyCont);
  sellOn := (sellFast or sellCont);
  bothOn := (buyOn and sellOn);

  // Viés científico de desempate:
  // 1) direção do momentum
  // 2) inclinação normalizada
  // 3) persistência da última entrada
  // 4) volume relativo e força de regime como reforço
  bias := 0;

  if (marketMomentum > 0.08) then
    bias := bias + 2
  else if (marketMomentum < -0.08) then
    bias := bias - 2;

  if (slopeN0 > 0.03) then
    bias := bias + 1
  else if (slopeN0 < -0.03) then
    bias := bias - 1;

  if (lastEntrySide = 1) then
    bias := bias + 1
  else if (lastEntrySide = -1) then
    bias := bias - 1;

  if (vrs0 >= vrsMinimo * 1.10) then
  begin
    if (marketMomentum >= 0) then
      bias := bias + 1
    else
      bias := bias - 1;
  end;

  if (regimeStrength >= (MinADX_eff + 4)) then
  begin
    if (slopeN0 >= 0) then
      bias := bias + 1
    else
      bias := bias - 1;
  end;

  // Gatilhos CANÔNICOS: nunca permite buyTrig e sellTrig simultâneos
  buyTrig  := False;
  sellTrig := False;

  if bothOn then
  begin
    if (bias > 0) then
      buyTrig := True
    else if (bias < 0) then
      sellTrig := True
    else
    begin
      // empate real: segue lado com maior coerência com o candle atual
      if (Close[0] >= Open[0]) then
        buyTrig := True
      else
        sellTrig := True;
    end;
  end
  else
  begin
    buyTrig  := buyOn;
    sellTrig := sellOn;
  end;

// --- PORTA DE EXECUÇÃO (Correção-Σ): SEMPRE PINTAR + XAI quando gate OFF ---
// Regra: nunca “sumir” com a pintura. Se gate OFF => NEUTRO + motivo.
// Novo código científico (ultra eficiente...) →
// Gate removido: ConfirmaCloseBB já é tratado no Confirm2Steps.
// O núcleo SEMPRE roda (SomenteSinais controla apenas execução/ordens, não a pintura).

// ===================== XAI CORE (reset por candle) =====================
xaiSignalReason   := "";

// reset do candle (ok)
xaiBlockReason := "";

xaiContext        := "";
xaiRegimeLabel    := "";
xaiVolLabel       := "";

xaiExitCause      := "";
xaiGuardNote      := "";
exitByConfidence  := False;
exitByReversal    := False;
exitByStop        := False;

    // CONTEXTO (Situation Awareness): Regime / Volatilidade / Horário / Gate
    // RegimeMercado: (0=tendência ADX) (1=normal) (2=tendência slope) (3=lateral BW/ATR)
    if      (regimeMercado = 0) then xaiRegimeLabel := "REG=TEND_ADX"
    else if (regimeMercado = 2) then xaiRegimeLabel := "REG=TEND_SLOPE"
    else if (regimeMercado = 3) then xaiRegimeLabel := "REG=LATERAL"
    else                              xaiRegimeLabel := "REG=NORMAL";

    // VolatilidadeState: 0=baixa | 1=média | 2=alta
    if      (volatilityState = 0) then xaiVolLabel := "VOL=BAIXA"
    else if (volatilityState = 2) then xaiVolLabel := "VOL=ALTA"
    else                                xaiVolLabel := "VOL=MEDIA";

    // Contexto final (sem conversão numérica -> evita parser e mantém custo baixo)
    if horarioPermitido then
      xaiContext := xaiRegimeLabel + "|" + xaiVolLabel + "|HR=OK"
    else
      xaiContext := xaiRegimeLabel + "|" + xaiVolLabel + "|HR=FORA";

    if adaptiveGateOk then
      xaiContext := xaiContext + "|GATE=OK"
    else
      xaiContext := xaiContext + "|GATE=OFF";

    // 1) FASE DE RESET NEUTRO PÓS-AMARELO (LEI CIENTÍFICA)
    if (resetBars > 0) then
    begin
      xaiBlockReason := "RESET_POS_AMARELO(1)";
      paintStateSeries[0] := 0;   // define estado neutro único (código unificado)

      resetBars := resetBars - 1;
      // (NTSL não possui Exit; e este IF/ELSE já bloqueia entradas/saídas nesta barra)
    end

    else
    begin

      // 2) FASE DE CAPTURA DE ENTRADAS (NEUROCIÊNCIA COMPUTACIONAL)
      if (not holdBuy) and (not holdSell) then
            begin
              // ANÁLISE MULTIDIMENSIONAL DO GATILHO (XAI/DSS)
              // Regra: se entrar (verde/vermelho) registrar "Motivo do Sinal".
              // Se NÃO entrar (neutro), registrar "Motivo do Bloqueio" (por que ficou de fora).

              if buyTrig and (not sellTrig) then
              begin
  
                // ATIVAÇÃO DO REGIME DE COMPRA
                holdBuy        := True;
                lastEntrySide  := 1;
                entryBarIdx    := CurrentBar();
                entryPrice     := Close[0];
                highSinceEntry := High[0];
                lowSinceEntry  := Low[0];
                hiBarIdx       := CurrentBar();
                barsDesdeSinal := 0;

                // ===== GESTÃO DIÁRIA VIVA =====
                // Em SomenteSinais, o "trade" é o próprio sinal.
                // Em automação, a contagem fica para o fill real.
                if SomenteSinais then
                begin
                  tradesHoje := tradesHoje + 1;
                  if (MaxTradesDia > 0) and (tradesHoje >= MaxTradesDia) then
                  begin
                    travaDia        := True;
                    auditProtReason := "MAXTRADES_DIA";
                  end;
                end;

                // REARME CIENTÍFICO DO STOP DO NOVO TRADE
                beArmed         := False;
                slPrice         := entryPrice - (SL_eff * atr0);
                stopTouchPx     := 0;
                stopTouchedBar  := False;

                // Reset correto dos ADAPTATIVOS (evita carry-over oculto entre trades)
                minGainTarget   := 0;
                minGainExitATR  := 0;
                epsStepTarget   := 0;
                epsStep         := 0;
                epsRetTarget    := 0;
                epsRet          := 0;

                // RESET CIENTÍFICO DO AMARELO
                // Impede que um AMARELO validado no trade anterior contamine o novo ciclo.
                optimalExit      := False;
                forceExit5m      := False;
                exitByStop       := False;
                exitByConfidence := False;
                exitByReversal   := False;

                // XAI (motivo da entrada)
                if buyFast then
                  xaiSignalReason := "ENTRADA_COMPRA:BUY_FAST"
                else if buyCont then
                  xaiSignalReason := "ENTRADA_COMPRA:BUY_CONT"
                else
                  xaiSignalReason := "ENTRADA_COMPRA:BUY_TRIG";

                auditEntryReason := xaiSignalReason;

                paintStateSeries[0] := 1;  // VERDE - ENTRADA COMPRADA
              end
   
              else if sellTrig and (not buyTrig) then
              begin
                // ATIVAÇÃO DO REGIME DE VENDA
                holdSell       := True;
                lastEntrySide  := -1;
                entryBarIdx    := CurrentBar();
                entryPrice     := Close[0];
                lowSinceEntry  := Low[0];
                highSinceEntry := High[0];
                loBarIdx       := CurrentBar();
                barsDesdeSinal := 0;

                // ===== GESTÃO DIÁRIA VIVA =====
                // Em SomenteSinais, o "trade" é o próprio sinal.
                // Em automação, a contagem fica para o fill real.
                if SomenteSinais then
                begin
                  tradesHoje := tradesHoje + 1;
                  if (MaxTradesDia > 0) and (tradesHoje >= MaxTradesDia) then
                  begin
                    travaDia        := True;
                    auditProtReason := "MAXTRADES_DIA";
                  end;
                end;

                // REARME CIENTÍFICO DO STOP DO NOVO TRADE
                beArmed         := False;
                slPrice         := entryPrice + (SL_eff * atr0);
                stopTouchPx     := 0;
                stopTouchedBar  := False;

                // Reset correto dos ADAPTATIVOS (evita carry-over oculto entre trades)
                minGainTarget   := 0;
                minGainExitATR  := 0;
                epsStepTarget   := 0;
                epsStep         := 0;
                epsRetTarget    := 0;
                epsRet          := 0;

                // RESET CIENTÍFICO DO AMARELO
                // Impede que um AMARELO validado no trade anterior contamine o novo ciclo.
                optimalExit      := False;
                forceExit5m      := False;
                exitByStop       := False;
                exitByConfidence := False;
                exitByReversal   := False;

                // XAI (motivo da entrada)
                if sellFast then
                  xaiSignalReason := "ENTRADA_VENDA:SELL_FAST"
                else if sellCont then
                  xaiSignalReason := "ENTRADA_VENDA:SELL_CONT"
                else
                  xaiSignalReason := "ENTRADA_VENDA:SELL_TRIG";

                auditEntryReason := xaiSignalReason;

                paintStateSeries[0] := -1; // VERMELHO - ENTRADA VENDIDA
              end

              else
              begin
                // XAI (motivo do bloqueio / por que ficou NEUTRO)
                // Flat sem trigger executável = NEUTRO explícito e canônico.
                // Isso evita herdar cor anterior por persistência implícita da série.
                paintStateSeries[0] := 0;

                // ==========================================================
                // GATE ÚNICO CIENTÍFICO (1 fonte de verdade, determinístico)
                // Objetivo: não bloquear lucro por conflito de flags
                // ==========================================================
                gateOpen       := True;
                xaiBlockReason := "";

                // 1) Hard stops (NUNCA negocia)
                // - travaDia/travaGanho: stops diários
                // - cooling: resfriamento do organismo
                // - auditProtActive: proteção anti-espiral (perdas seguidas/expectancy)
                allowTradeGlobal := (not travaDia) and (not travaGanho) and (cooling <= 0) and (not auditProtActive);

                if (not allowTradeGlobal) then
                begin
                  gateOpen := False;

                  // Prioridade XAI: se já existe motivo, ele é a verdade
                  if (auditProtReason <> "") then
                    xaiBlockReason := auditProtReason
                  else if travaDia then
                    xaiBlockReason := "STOP_DIA"
                  else if travaGanho then
                    xaiBlockReason := "STOP_DIA_GAIN"
                  else if (cooling > 0) then
                    xaiBlockReason := "COOLDOWN_LOSS"
                  else
                    xaiBlockReason := "PROTECT_AUDITOR";
                end
                else
                begin
                  // 2) Regras de consistência (evita sinais “impossíveis”)
                  if (buyTrig and sellTrig) then
                  begin
                    gateOpen       := False;
                    xaiBlockReason := "BLOCK:CONFLITO_BUY_SELL";
                  end
                  // 3) Reset neutro pós-amarelo (regra de transição limpa)
                  else if (resetBars > 0) then
                  begin
                    gateOpen       := False;
                    xaiBlockReason := "BLOCK:RESET_POS_AMARELO";
                  end
                  // 4) Podas locais CONSOLIDADAS (ERA CALCULADO, MAS NÃO ENTRAVA NO GATE)
                  // - allowTrade já incorpora: janela morta, filtros de congestão/breakout, podas por range etc.
                  // - Agora vira parte do GATE ÚNICO (sem duplicar lógica em outros lugares)
                  else if (not allowTrade) then
                  begin
                    gateOpen       := False;
                    xaiBlockReason := "BLOCK:ALLOWTRADE_LOCAL";
                  end
                  // 5) Filtros operacionais (podas inteligentes)
                  else if (not candleOk) then
                  begin
                    gateOpen       := False;
                    xaiBlockReason := "BLOCK:CANDLE_NAO_OK";
                  end
                  else if (not horarioPermitido) then
                  begin
                    gateOpen       := False;
                    xaiBlockReason := "BLOCK:HORARIO";
                  end
                  // 6) Gate adaptativo (último da fila para não “matar” oportunidade por ruído)
                  else if (not adaptiveGateOk) then
                  begin
                    gateOpen       := False;
                    xaiBlockReason := "BLOCK:ADAPTIVE_GATE";
                  end;
                end;

                // (Importante) não duplica verificação: gateOpen/xaiBlockReason já são a autoridade
              end;
          end

          else
          begin

        // 3) DETECÇÃO DO PONTO ÓTIMO COM MÚLTIPLAS CAMADAS DE CONFIRMAÇÃO
        if (holdBuy or holdSell) then
        begin
          // ATUALIZAÇÃO DO CONTADOR DE BARRAS (CRÍTICO!)
          barsDesdeSinal := barsDesdeSinal + 1;

          // Snapshot seguro de variáveis críticas (Manual 4.5)
          currentHigh := High[0];
          currentLow := Low[0];
          currentBody := Abs(Close[0] - Open[0]);
          currentRange := Max(0.0001, currentHigh - currentLow);

          // I. CÁLCULO DE GANHO MÁXIMO NORMALIZADO (FÍSICA FINANCEIRA)
          if holdBuy then
          begin
            // ATUALIZA MÁXIMA DO TRADE
            if currentHigh > highSinceEntry then
            begin
              highSinceEntry := currentHigh;
              hiBarIdx := CurrentBar();
            end;

            maxGainATR := (highSinceEntry - entryPrice) / Max(0.0001, atr0);
            currentGainATR := (currentHigh - entryPrice) / Max(0.0001, atr0);

            // Wick superior normalizado (exaustão)
            upperWickCurrent := (currentHigh - Max(Open[0], Close[0])) / Max(0.0001, currentRange);
            
            // II. DETECÇÃO CIENTÍFICA DO AMARELO (HIERÁRQUICA, SEM CONFLITO, EXECUTÁVEL)
            // Prioridade:
            // A) Topo real + lucro mínimo (determinístico)
            // B) Exaustão (wick) + (banda OU vol alta não exige banda) + lucro razoável
            // C) Score XAI como fallback (não manda na saída se já há topo real)
            // D) MFE + retração em ATR (sem duplicidade, wick-aware)
            // E) Anti-ruído em vol alta (evita amarelo por spike/noise)

            // Normalizações leves (sem criar variáveis novas)
            bodyATR := currentBody / Max(0.0001, atr0);

            // =====================================================
            // AMARELO DA COMPRA — MODO BAIXO RISCO / CLÁUSULA PÉTREA
            // Objetivo: sair mais cedo do que o proxy 5m tardio,
            // preservando lucro e sem quebrar o restante do sistema.
            // Lógica:
            // 1) o trade precisa ter lucro mínimo real;
            // 2) precisa haver devolução material do topo;
            // 3) precisa haver reversão estrutural moderada;
            // 4) não cria função nova, não cria conflito, não muda a venda.
            // =====================================================
            forceExit5m := False;

            // lucro corrente e devolução do topo em ATR
            pnlCurrent  := (Close[0] - entryPrice) / Max(0.0001, atr0);
            retrace5ATR := (highSinceEntry - Min(Close[0], currentLow)) / Max(0.0001, atr0);

            // slope curto e causal, sem esperar 5 candles completos
            lagIdx := 2;
            if (CurrentBar() < lagIdx) then
              lagIdx := CurrentBar();

            if (lagIdx >= 1) then
              slope5N := (Close[0] - Close[lagIdx]) / Max(0.0001, (lagIdx * 1.0) * atr0)
            else
              slope5N := 0.0;                    

            // limiares adaptativos — compra, supervisionados pelo regime + auditor
            minProfitATR := 0.13 + (volatilityState * 0.025);
            epsRet       := 0.13 + (volatilityState * 0.025);

            if (regimeMercado = 3) then
            begin
              minProfitATR := minProfitATR - 0.03;
              epsRet       := epsRet - 0.02;
            end
            else if (regimeMercado = 2) then
            begin
              minProfitATR := minProfitATR + 0.01;
              epsRet       := epsRet + 0.01;
            end;

            if (barsDesdeSinal >= 4) then
              epsRet := epsRet - 0.015;

            candidateFloor := 0.08 + (volatilityState * 0.015);
            candidateCeil  := 0.22 + Min(0.04, regimeStrength / 500.0);

            if (auditExpectancy_ATR > 0) then
              candidateFloor := Max(0.07, candidateFloor - Min(0.015, auditExpectancy_ATR * 0.025))
            else
              candidateFloor := candidateFloor + Min(0.02, Abs(auditExpectancy_ATR) * 0.025);

            if auditProtActive then
              candidateFloor := candidateFloor + 0.015;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            minProfitATR := Max(candidateFloor, Min(candidateCeil, minProfitATR));
            epsRet       := Max(0.07, Min(0.28, epsRet + Max(0.0, regimeAdjExit * 0.16)));

             // 1) Topo real EXECUTÁVEL do trade (sem repaint / sem olhar futuro)
            // Correção-Σ:
            // - P0: pinta AMARELO no próprio candle de máxima quando já existe exaustão mensurável.
            // - P1: se a reversão só fica clara no candle seguinte, sai no 1º candle realmente executável.
            // - P2: trava lucro se houver devolução material antes de virar prejuízo.
            offsetMaxima := CurrentBar() - hiBarIdx;
            if (offsetMaxima < 0) then
              offsetMaxima := 0;

            condition1 := (offsetMaxima = 0);

            // Wick dinâmico: mais permissivo em lateralidade, mais exigente em volatilidade alta.
            ratioThresh := 0.28 + (volatilityState * 0.04);
            if (regimeMercado = 3) then
              ratioThresh := Max(0.23, ratioThresh - 0.03);

            condition2 := (upperWickCurrent >= ratioThresh);

            // Extremo técnico: banda, volatilidade ou MFE suficiente.
            condition3 := (currentHigh >= bbTop[0]) or
                          (Close[0] >= bbTop[0]) or
                          (volatilityState = 2) or
                          (maxGainATR >= Max(0.16, minGainExitATR * 0.50));

            // Reversão causal, sempre usando somente dados já formados.
            condition4 := False;
            condition5 := False;

            if CurrentBar() >= 1 then
            begin
              condition4 :=
                (Close[0] < Close[1]) or
                (currentLow <= Low[1]) or
                (Close[0] < Open[0]) or
                (Close[0] <= ((currentHigh + currentLow) * 0.50));

              condition5 :=
                (slope5N <= -(0.006 + (volatilityState * 0.002))) or
                ((slopeN[0] < slopeN[1]) and (ifrValue[0] <= ifrValue[1])) or
                (adx0 <= (adxPrev - (0.25 + (volatilityState * 0.05)))) or
                ((vrs0 < vrsMedia0) and (Close[0] <= Close[1]));
            end;

            // Lucro mínimo líquido adaptativo: baixo risco, mas não hiperprecoce.
            minProfitATR := 0.11 + (volatilityState * 0.02);
            if (regimeMercado = 3) then
              minProfitATR := minProfitATR - 0.02
            else if (regimeMercado = 2) then
              minProfitATR := minProfitATR + 0.01;

            candidateFloor := Max(0.07, costFloorATR + 0.015);
            candidateCeil  := 0.24 + Min(0.05, regimeStrength / 420.0);

            if (auditExpectancy_ATR > 0) then
              candidateFloor := Max(0.06, candidateFloor - Min(0.012, auditExpectancy_ATR * 0.020))
            else
              candidateFloor := candidateFloor + Min(0.018, Abs(auditExpectancy_ATR) * 0.020);

            if auditProtActive then
              candidateFloor := candidateFloor + 0.015;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            minProfitATR := Max(candidateFloor, Min(candidateCeil, minProfitATR));

            currentProfitATR := (Close[0] - entryPrice) / Max(0.0001, atr0);
            retrace5ATR      := (highSinceEntry - Min(Close[0], currentLow)) / Max(0.0001, atr0);

            // Mantém o organismo autoajustável sem salto rígido.
            minGainTarget := 0.30 + (volatilityState * 0.06);
            if (regimeMercado = 3) then
              minGainTarget := minGainTarget - 0.04
            else if (regimeMercado = 2) then
              minGainTarget := minGainTarget + 0.02;
            if (barsDesdeSinal >= 6) then
              minGainTarget := minGainTarget - Min(0.05, (barsDesdeSinal - 5) * 0.008);
            if (Abs(marketMomentum) > 0.45) then
              minGainTarget := minGainTarget + 0.02
            else if (Abs(marketMomentum) < 0.15) then
              minGainTarget := minGainTarget - 0.02;

            minGainTarget  := Max(candidateFloor, Min(Max(candidateCeil, 0.18), minGainTarget));
            minGainExitATR := minGainExitATR + 0.34 * (minGainTarget - minGainExitATR);
            condition5     := condition5 or (maxGainATR >= minGainExitATR);

            // P0Σ — PPP TOP EXECUTÁVEL / COMPRA (CORREÇÃO-Σ ANTI-ATRASO)
            // Objetivo:
            // 1) permitir AMARELO no candle da máxima executável;
            // 2) não exigir reversão completa tardia;
            // 3) aceitar topo limpo quando o MFE líquido já compensa custo/risco;
            // 4) manter PaintBar 100% centralizado no Color Engine.
            if (not optimalExit) and (atr0 > 0) and (offsetMaxima = 0) then
            begin
              pnlATR           := (highSinceEntry - entryPrice) / Max(0.0001, atr0);
              currentProfitATR := (Close[0] - entryPrice) / Max(0.0001, atr0);

              if (currentProfitATR >= Max(costFloorATR + Capture_MinNetATR, minProfitATR * 0.85)) and
                 (pnlATR >= Max(costFloorATR + Capture_MinNetATR,
                                Max(0.20 + (volatilityState * 0.025), minProfitATR * 1.05))) and
                 (
                   (upperWickCurrent >= Max(0.18, ratioThresh * 0.70)) or
                   (Close[0] <= (currentHigh - Max(0.05 * atr0, 0.08 * currentRange))) or
                   (Close[0] <= ((currentHigh + currentLow) * 0.58)) or
                   (pnlATR >= Max(minGainExitATR * 0.88,
                                  Max(costFloorATR + Capture_MinNetATR, minProfitATR * 1.18)))
                 ) and
                 (barsDesdeSinal >= 1) and
                 (not doji) then
              begin
                forceExit5m    := True;
                optimalExit    := True;
                exitByReversal := True;

                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=PPP_TOP_ANTIDELAY_BUY";

                yellowTheoreticalPx := highSinceEntry;
                yellowVisualPx      := Close[0];
                auditExitPx         := Close[0];
                auditExitBar        := CurrentBar();
              end;
            end

            // P1 — refinamento cirúrgico: 1ª reversão após o topo executável.
            // Não espera devolução grande: exige MFE líquido + fechamento ainda viável.
            else if (not optimalExit) and
                    (CurrentBar() >= 1) and
                    (offsetMaxima = 1) and
                    (maxGainATR >= Max(costFloorATR + Capture_MinNetATR, 0.15)) and
                    (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.80)) and
                    (
                      (Close[0] < Close[1]) or
                      (currentLow <= Low[1]) or
                      (Close[0] < Open[0]) or
                      (Close[0] <= ((currentHigh + currentLow) * 0.50))
                    ) then
            begin
              forceExit5m    := True;
              optimalExit    := True;
              exitByReversal := True;
              if (xaiExitCause = "") then xaiExitCause := "EXIT=BUY_REVERSAL_EDGE_EXEC";
            end

            // P2 — trava lucro quando a devolução já ficou material, sem esperar sinal oposto.
            else if (not optimalExit) and
                    (currentProfitATR >= Max(costFloorATR, minProfitATR * 0.25)) and
                    (retrace5ATR >= Max(0.045, minProfitATR * 0.40)) and
                    (condition4 or condition5) then
            begin
              forceExit5m    := True;
              optimalExit    := True;
              exitByReversal := True;
              if (xaiExitCause = "") then xaiExitCause := "EXIT=POST_TOP_CONFIRM_EXEC";
            end;

            // AMARELO PRIORITÁRIO: EXTREMO + MICRO-REVERSÃO (COMPRA)
            // =====================================================
            // Versão supervisionada: a base nasce do regime + auditor + tempo de trade.
            // Os guardrails continuam existindo, mas agora entram como LIMITADORES
            // de uma trajetória gradual, não como clamps rígidos cegos.
            epsStepTarget := 0.40 + (volatilityState * 0.04) + Max(0.0, regimeAdjExit * 0.30);
            if (regimeMercado = 0) then
              epsStepTarget := epsStepTarget + 0.06
            else if (regimeMercado = 2) then
              epsStepTarget := epsStepTarget + 0.03
            else if (regimeMercado = 3) then
              epsStepTarget := epsStepTarget - 0.08;
            if (barsDesdeSinal >= 6) then
              epsStepTarget := epsStepTarget - Min(0.05, (barsDesdeSinal - 5) * 0.008);
            if (bodyATR >= 0.35) then
              epsStepTarget := epsStepTarget - 0.02;
            if (auditExpectancy_ATR < 0) then
              epsStepTarget := epsStepTarget + Min(0.06, Abs(auditExpectancy_ATR) * 0.05)
            else
              epsStepTarget := epsStepTarget - Min(0.03, auditExpectancy_ATR * 0.04);
            if (auditProfitFactor < 1.0) then
              epsStepTarget := epsStepTarget + Min(0.05, (1.0 - auditProfitFactor) * 0.08)
            else
              epsStepTarget := epsStepTarget - Min(0.02, (auditProfitFactor - 1.0) * 0.04);
            candidateFloor := 0.12 + (volatilityState * 0.02) + Max(0.0, regimeAdjExit * 0.10);
            candidateCeil  := 0.92 - Min(0.18, regimeStrength / 220.0);
            epsStep := Max(candidateFloor, Min(candidateCeil, epsStepTarget));


            if auditProtActive then
              candidateFloor := candidateFloor + 0.04;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            epsStepTarget := Max(candidateFloor, Min(candidateCeil, epsStepTarget));

            adaptAlpha := 0.38;
            if auditProtActive then
              adaptAlpha := 0.20
            else if (volatilityState = 2) then
              adaptAlpha := 0.28
            else if (Abs(epsStepTarget - epsStep) > 0.10) then
              adaptAlpha := 0.48;

            if (epsStep <= 0) then
              epsStep := epsStepTarget
            else
              epsStep := epsStep + adaptAlpha * (epsStepTarget - epsStep);

            epsStep := Max(candidateFloor, Min(candidateCeil, epsStep));

            epsRetTarget := 0.10 + (volatilityState * 0.025) + Max(0.0, regimeAdjExit * 0.16);
            if (regimeMercado = 3) then
              epsRetTarget := epsRetTarget - 0.03
            else if (regimeMercado = 2) then
              epsRetTarget := epsRetTarget + 0.03;
            if (bodyATR >= 0.35) then
              epsRetTarget := epsRetTarget - 0.02;
            if (barsDesdeSinal >= 6) then
              epsRetTarget := epsRetTarget - Min(0.03, (barsDesdeSinal - 5) * 0.006);
            candidateFloor := 0.07 + (volatilityState * 0.015);
            candidateCeil  := 0.34 + Max(0.0, regimeAdjExit * 0.18);
            epsRet := Max(candidateFloor, Min(candidateCeil, epsRetTarget));

            // =====================================================
            if (not optimalExit) and (atr0 > 0) then
            begin
              pnlATR  := (highSinceEntry - entryPrice) / Max(0.0001, atr0);
              moveATR := (highSinceEntry - Min(Close[0], currentLow)) / Max(0.0001, atr0);

              if (pnlATR >= epsStep) and
                 (
                   // Caminho clássico: já houve devolução/micro-reversão suficiente.
                   (moveATR >= epsRet) or

                   // Caminho aprimorado: exaustão executável por pavio + VRS.
                   (
                     (upperWickCurrent >= ratioThresh) and
                     (vrs0 >= Max(0.0001, vrsMedia0) * (0.92 + (volatilityState * 0.04))) and
                     (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.80))
                   )
                 ) then
              begin
                optimalExit    := True;
                exitByReversal := True;

                if (
                     (upperWickCurrent >= ratioThresh) and
                     (vrs0 >= Max(0.0001, vrsMedia0) * (0.92 + (volatilityState * 0.04))) and
                     (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.80))
                   ) then
                begin
                  forceExit5m := True;

                  if (xaiExitCause = "") then
                    xaiExitCause := "EXIT=EXHAUST_WICK_VRS_TOP_BUY";
                end
                else
                begin
                  if (xaiExitCause = "") then
                    xaiExitCause := "EXIT=EXTREME_MICROREV_BUY";
                end;
              end;
            end;

            // =====================================================
            // SCORE XAI COMO FALLBACK REAL
            // =====================================================
            confidenceScore := 0;
            if condition1 then confidenceScore := confidenceScore + 25;
            if condition2 then confidenceScore := confidenceScore + 30;
            if condition3 then confidenceScore := confidenceScore + 20;
            if condition4 then confidenceScore := confidenceScore + 20;
            if condition5 then confidenceScore := confidenceScore + 15;

            confidenceThreshold := 46 + (volatilityState * 5) + Max(0.0, regimeAdjExit * 8);

            if (regimeMercado = 0) then
              confidenceThreshold := confidenceThreshold + 4
            else if (regimeMercado = 2) then
              confidenceThreshold := confidenceThreshold + 2
            else if (regimeMercado = 3) then
              confidenceThreshold := confidenceThreshold - 3;

            confidenceThreshold := confidenceThreshold - Min(14, barsDesdeSinal * 1.4);

            if (maxGainATR >= minGainExitATR) then
              confidenceThreshold := confidenceThreshold - 6;

            if (auditExpectancy_ATR < 0) then
              confidenceThreshold := confidenceThreshold + Min(8, Abs(auditExpectancy_ATR) * 6)
            else
              confidenceThreshold := confidenceThreshold - Min(4, auditExpectancy_ATR * 4);

            if (auditProfitFactor < 1.0) then
              confidenceThreshold := confidenceThreshold + Min(6, (1.0 - auditProfitFactor) * 10)
            else
              confidenceThreshold := confidenceThreshold - Min(3, (auditProfitFactor - 1.0) * 4);

            if (auditEwmaWin < 0.50) then
              confidenceThreshold := confidenceThreshold + Min(6, (0.50 - auditEwmaWin) * 18)
            else
              confidenceThreshold := confidenceThreshold - Min(3, (auditEwmaWin - 0.50) * 10);

            candidateFloor := 24 + (volatilityState * 2) + Max(0.0, regimeAdjExit * 6);
            candidateCeil  := 82 - Min(6, regimeStrength / 18.0);

            if auditProtActive then
              candidateFloor := candidateFloor + 4;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            confidenceThreshold := Max(candidateFloor, Min(candidateCeil, confidenceThreshold));

            // =====================================================
            confidenceAmarelo := (confidenceScore >= confidenceThreshold);

            if (not optimalExit) and (atr0 > 0) then
            begin
              pnlATR  := (highSinceEntry - entryPrice) / atr0;
              moveATR := (highSinceEntry - Min(Close[0], currentLow)) / atr0;

              if confidenceAmarelo and
                 (pnlATR >= Max(minGainExitATR * 0.76, epsStep * 0.74)) and
                 (condition3 or (offsetMaxima <= 1)) and
                 (moveATR >= (epsRet * 0.45)) then
              begin
                optimalExit := True;
                if (xaiExitCause = "") then xaiExitCause := "EXIT=CONFIRM_XAI_AFTER_EXTREME_BUY";
              end;
            end;

            // =====================================================
            // PROTEÇÃO CONTRA AMARELO PRECOCE EM VOL ALTA
            // Correção-Σ:
            // - continua filtrando ruído verdadeiro;
            // - mas NÃO revoga um AMARELO já validado quando já existe
            //   reversão executável + lucro líquido mínimo acima do piso de custo;
            // - em atraso > 1 barra após o topo, a prioridade passa a ser
            //   travar lucro, não insistir em permanência.
            // =====================================================
            if (volatilityState = 2) and (optimalExit) and (not forceExit5m) and (not exitByStop) and
              (offsetMaxima > 1) then
            begin
              pnlATR  := (highSinceEntry - entryPrice) / Max(0.0001, atr0);
              moveATR := (highSinceEntry - Min(Close[0], currentLow)) / Max(0.0001, atr0);
              candidateFloor := Max(0.22, Min(0.48, minGainExitATR * 0.78));

              // Se já há lucro líquido mínimo e reversão causal executável,
              // o AMARELO deve sobreviver.
              if (pnlATR >= costFloorATR) and
                 (moveATR >= Max(0.05, epsRet * 0.40)) and
                 (
                   (Close[0] < Close[1]) or
                   (currentLow <= Low[1]) or
                   (Close[0] < Open[0])
                 ) then
              begin
                optimalExit := True;

                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=POST_TOP_CONFIRM_EXEC";
              end

              else if (xaiExitCause <> "EXIT=EXTREME_MICROREV_BUY") and
                      (xaiExitCause <> "EXIT=BUY_REVERSAL_LOW_RISK") and
                      (xaiExitCause <> "EXIT=BUY_REVERSAL_EDGE_EXEC") and
                      (xaiExitCause <> "EXIT=STRUCT_REV_AFTER_TOP_EXEC") and
                      (xaiExitCause <> "EXIT=POST_TOP_CONFIRM_EXEC") and
                      (xaiExitCause <> "EXIT=CONFIRM_XAI_AFTER_EXTREME_BUY") and
                      (xaiExitCause <> "EXIT=TIME_DECAY_PROFIT_LOCK_BUY") and
                      (xaiExitCause <> "EXIT=EXHAUST_WICK(+BB)+PROFIT_OK") then
              begin
                if (bodyATR < 0.12) and
                   (moveATR < (epsRet * 0.72)) and
                   (pnlATR < Max(candidateFloor, epsStep + 0.08)) then
                begin
                  optimalExit := False;
                  xaiExitCause := "BLOCK:VOL_NOISE_WEAKREV_BUY";
                end;

                if optimalExit and
                   ((currentHigh - Max(Open[0], Close[0])) > (0.72 * currentRange)) and
                   (moveATR < (epsRet * 0.92)) and
                   (pnlATR < Max(candidateFloor, minGainExitATR * 0.88)) then
                begin
                  optimalExit := False;
                  xaiExitCause := "BLOCK:VOL_SPIKE_CONT_BUY";
                end;

                if optimalExit and condition3 and
                   (Close[0] < (bbTop[0] - 0.08 * atr0)) and
                   (moveATR < (epsRet * 0.92)) and
                   (pnlATR < Max(candidateFloor, minGainExitATR * 0.88)) then
                begin
                  optimalExit := False;
                  xaiExitCause := "BLOCK:VOL_LEVELTEST_WEAKREV_BUY";
                end;
              end;

              if (not optimalExit) then
              begin
                // Correção-Σ:
                // Não reposicionar stop usando informação intrabar já conhecida.
                // Isso evita stop retroativo/falso no mesmo candle.
                // A proteção real continua soberana no ProfitGuardian/trailing já calculado.
                if (xaiGuardNote = "") then
                  xaiGuardNote := "PG=VOL_HIGH_NO_RETRO_STOP_BUY";
              end;
            end;
          end

          else if holdSell then
          begin
            // ATUALIZA MÍNIMA DO TRADE
            if currentLow < lowSinceEntry then
            begin
              lowSinceEntry := currentLow;
              loBarIdx := CurrentBar();
            end;

            maxGainATR     := (entryPrice - lowSinceEntry) / Max(0.0001, atr0);
            currentGainATR := (entryPrice - currentLow)     / Max(0.0001, atr0);

            // Wick inferior normalizado (exaustão)            
            lowerWickCurrent := (Min(Open[0], Close[0]) - currentLow) / Max(0.0001, currentRange);

            // Normalizações leves (sem criar variáveis novas)
            bodyATR := currentBody / Max(0.0001, atr0);     

            // =====================================================
            // AMARELO DA VENDA — MODO BAIXO RISCO / CLÁUSULA PÉTREA
            // Objetivo: sair mais cedo do que o proxy 5m tardio,
            // preservando lucro e sem quebrar o restante do sistema.
            // Lógica:
            // 1) a venda precisa ter lucro mínimo real;
            // 2) precisa haver devolução material do fundo;
            // 3) precisa haver reversão estrutural moderada;
            // 4) não cria função nova, não cria conflito, não muda a compra.
            // =====================================================
             forceExit5m := False;

            // lucro corrente e devolução do fundo em ATR
            pnlCurrent  := (entryPrice - Close[0]) / Max(0.0001, atr0);
            retrace5ATR := (Max(Close[0], currentHigh) - lowSinceEntry) / Max(0.0001, atr0);

            // slope curto e causal, sem esperar 5 candles completos
            lagIdx := 2;
            if (CurrentBar() < lagIdx) then
              lagIdx := CurrentBar();

            if (lagIdx >= 1) then
              slope5N := (Close[0] - Close[lagIdx]) / Max(0.0001, (lagIdx * 1.0) * atr0)
            else
              slope5N := 0.0;

            // limiares adaptativos — venda, supervisionados pelo regime + auditor
            minProfitATR := 0.13 + (volatilityState * 0.025);
            epsRet       := 0.13 + (volatilityState * 0.025);

            if (regimeMercado = 3) then
            begin
              minProfitATR := minProfitATR - 0.03;
              epsRet       := epsRet - 0.02;
            end
            else if (regimeMercado = 2) then
            begin
              minProfitATR := minProfitATR + 0.01;
              epsRet       := epsRet + 0.01;
            end;

            if (barsDesdeSinal >= 4) then
              epsRet := epsRet - 0.015;

            candidateFloor := 0.08 + (volatilityState * 0.015);
            candidateCeil  := 0.22 + Min(0.04, regimeStrength / 500.0);

            if (auditExpectancy_ATR > 0) then
              candidateFloor := Max(0.07, candidateFloor - Min(0.015, auditExpectancy_ATR * 0.025))
            else
              candidateFloor := candidateFloor + Min(0.02, Abs(auditExpectancy_ATR) * 0.025);

            if auditProtActive then
              candidateFloor := candidateFloor + 0.015;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            minProfitATR := Max(candidateFloor, Min(candidateCeil, minProfitATR));
            epsRet       := Max(0.07, Min(0.28, epsRet + Max(0.0, regimeAdjExit * 0.16)));

            // 1) Fundo real EXECUTÁVEL do trade (sem repaint / sem olhar futuro)
            // Correção-Σ:
            // - P0: pinta AMARELO no próprio candle de mínima quando já existe exaustão mensurável.
            // - P1: se o repique só fica claro no candle seguinte, sai no 1º candle realmente executável.
            // - P2: trava lucro se houver devolução material antes de virar prejuízo.
            offsetMinima := CurrentBar() - loBarIdx;
            if (offsetMinima < 0) then
              offsetMinima := 0;

            condition1 := (offsetMinima = 0);

            // Wick dinâmico correto para SELL = pavio inferior.
            ratioThresh := 0.28 + (volatilityState * 0.04);
            if (regimeMercado = 3) then
              ratioThresh := Max(0.23, ratioThresh - 0.03);

            condition2 := (lowerWickCurrent >= ratioThresh);

            // Extremo técnico: banda, volatilidade ou MFE suficiente.
            condition3 := (currentLow <= bbBot[0]) or
                          (Close[0] <= bbBot[0]) or
                          (volatilityState = 2) or
                          (maxGainATR >= Max(0.16, minGainExitATR * 0.50));

            // Reversão causal, sempre usando somente dados já formados.
            condition4 := False;
            condition5 := False;

            if CurrentBar() >= 1 then
            begin
              condition4 :=
                (Close[0] > Close[1]) or
                (currentHigh >= High[1]) or
                (Close[0] > Open[0]) or
                (Close[0] >= ((currentHigh + currentLow) * 0.50));

              condition5 :=
                (slope5N >= (0.006 + (volatilityState * 0.002))) or
                ((slopeN[0] > slopeN[1]) and (ifrValue[0] >= ifrValue[1])) or
                (adx0 <= (adxPrev - (0.25 + (volatilityState * 0.05)))) or
                ((vrs0 < vrsMedia0) and (Close[0] >= Close[1]));
            end;

            // Lucro mínimo líquido adaptativo: baixo risco, mas não hiperprecoce.
            minProfitATR := 0.11 + (volatilityState * 0.02);
            if (regimeMercado = 3) then
              minProfitATR := minProfitATR - 0.02
            else if (regimeMercado = 2) then
              minProfitATR := minProfitATR + 0.01;

            candidateFloor := Max(0.07, costFloorATR + 0.015);
            candidateCeil  := 0.24 + Min(0.05, regimeStrength / 420.0);

            if (auditExpectancy_ATR > 0) then
              candidateFloor := Max(0.06, candidateFloor - Min(0.012, auditExpectancy_ATR * 0.020))
            else
              candidateFloor := candidateFloor + Min(0.018, Abs(auditExpectancy_ATR) * 0.020);

            if auditProtActive then
              candidateFloor := candidateFloor + 0.015;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            minProfitATR := Max(candidateFloor, Min(candidateCeil, minProfitATR));

            currentProfitATR := (entryPrice - Close[0]) / Max(0.0001, atr0);
            retrace5ATR      := (Max(Close[0], currentHigh) - lowSinceEntry) / Max(0.0001, atr0);

            // Mantém o organismo autoajustável sem salto rígido.
            minGainTarget := 0.30 + (volatilityState * 0.06);
            if (regimeMercado = 3) then
              minGainTarget := minGainTarget - 0.04
            else if (regimeMercado = 2) then
              minGainTarget := minGainTarget + 0.02;
            if (barsDesdeSinal >= 6) then
              minGainTarget := minGainTarget - Min(0.05, (barsDesdeSinal - 5) * 0.008);
            if (Abs(marketMomentum) > 0.45) then
              minGainTarget := minGainTarget + 0.02
            else if (Abs(marketMomentum) < 0.15) then
              minGainTarget := minGainTarget - 0.02;

            minGainTarget  := Max(candidateFloor, Min(Max(candidateCeil, 0.18), minGainTarget));
            minGainExitATR := minGainExitATR + 0.34 * (minGainTarget - minGainExitATR);
            condition5     := condition5 or (maxGainATR >= minGainExitATR);

            // P0Σ — PPP BOTTOM EXECUTÁVEL / VENDA (CORREÇÃO-Σ ANTI-ATRASO)
            // Objetivo:
            // 1) permitir AMARELO no candle da mínima executável;
            // 2) não exigir repique completo tardio;
            // 3) aceitar fundo limpo quando o MFE líquido já compensa custo/risco;
            // 4) manter PaintBar 100% centralizado no Color Engine.
            if (not optimalExit) and (atr0 > 0) and (offsetMinima = 0) then
            begin
              pnlATR           := (entryPrice - lowSinceEntry) / Max(0.0001, atr0);
              currentProfitATR := (entryPrice - Close[0]) / Max(0.0001, atr0);

              if (currentProfitATR >= Max(costFloorATR + Capture_MinNetATR, minProfitATR * 0.85)) and
                 (pnlATR >= Max(costFloorATR + Capture_MinNetATR,
                                Max(0.20 + (volatilityState * 0.025), minProfitATR * 1.05))) and
                 (
                   (lowerWickCurrent >= Max(0.18, ratioThresh * 0.70)) or
                   (Close[0] >= (currentLow + Max(0.05 * atr0, 0.08 * currentRange))) or
                   (Close[0] >= ((currentHigh + currentLow) * 0.42)) or
                   (pnlATR >= Max(minGainExitATR * 0.88,
                                  Max(costFloorATR + Capture_MinNetATR, minProfitATR * 1.18)))
                 ) and
                 (barsDesdeSinal >= 1) and
                 (not doji) then
              begin
                forceExit5m    := True;
                optimalExit    := True;
                exitByReversal := True;

                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=PPP_BOTTOM_ANTIDELAY_SELL";

                yellowTheoreticalPx := lowSinceEntry;
                yellowVisualPx      := Close[0];
                auditExitPx         := Close[0];
                auditExitBar        := CurrentBar();
              end;
            end

            // P1 — refinamento cirúrgico: 1º repique após o fundo executável.
            // Não espera devolução grande: exige MFE líquido + fechamento ainda viável.
            else if (not optimalExit) and
                    (CurrentBar() >= 1) and
                    (offsetMinima = 1) and
                    (maxGainATR >= Max(costFloorATR + Capture_MinNetATR, 0.15)) and
                    (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.80)) and
                    (
                      (Close[0] > Close[1]) or
                      (currentHigh >= High[1]) or
                      (Close[0] > Open[0]) or
                      (Close[0] >= ((currentHigh + currentLow) * 0.50))
                    ) then
            begin
              forceExit5m    := True;
              optimalExit    := True;
              exitByReversal := True;
              if (xaiExitCause = "") then xaiExitCause := "EXIT=SELL_REVERSAL_EDGE_EXEC";
            end

            // P2 — trava lucro quando a devolução já ficou material, sem esperar sinal oposto.
            else if (not optimalExit) and
                    (currentProfitATR >= Max(costFloorATR, minProfitATR * 0.25)) and
                    (retrace5ATR >= Max(0.045, minProfitATR * 0.40)) and
                    (condition4 or condition5) then
            begin
              forceExit5m    := True;
              optimalExit    := True;
              exitByReversal := True;
              if (xaiExitCause = "") then xaiExitCause := "EXIT=POST_BOTTOM_CONFIRM_EXEC";
            end;

            // =====================================================
            // AMARELO PRIORITÁRIO: EXTREMO + MICRO-REVERSÃO (VENDA)
            // Regra: o preço manda primeiro; XAI confirma depois.
            // Objetivo: capturar o fundo útil com menos atraso, sem quebrar determinismo.
            // =====================================================
            epsStepTarget := 0.40 + (volatilityState * 0.04) + Max(0.0, regimeAdjExit * 0.30);
            if (regimeMercado = 0) then
              epsStepTarget := epsStepTarget + 0.06
            else if (regimeMercado = 2) then
              epsStepTarget := epsStepTarget + 0.03
            else if (regimeMercado = 3) then
              epsStepTarget := epsStepTarget - 0.08;
            if (barsDesdeSinal >= 6) then
              epsStepTarget := epsStepTarget - Min(0.05, (barsDesdeSinal - 5) * 0.008);
            if (bodyATR >= 0.35) then
              epsStepTarget := epsStepTarget - 0.02;
            if (auditExpectancy_ATR < 0) then
              epsStepTarget := epsStepTarget + Min(0.06, Abs(auditExpectancy_ATR) * 0.05)
            else
              epsStepTarget := epsStepTarget - Min(0.03, auditExpectancy_ATR * 0.04);
            if (auditProfitFactor < 1.0) then
              epsStepTarget := epsStepTarget + Min(0.05, (1.0 - auditProfitFactor) * 0.08)
            else
              epsStepTarget := epsStepTarget - Min(0.02, (auditProfitFactor - 1.0) * 0.04);
            candidateFloor := 0.12 + (volatilityState * 0.02) + Max(0.0, regimeAdjExit * 0.10);
            candidateCeil  := 0.92 - Min(0.18, regimeStrength / 220.0);
            epsStep := Max(candidateFloor, Min(candidateCeil, epsStepTarget));
  
            if auditProtActive then
              candidateFloor := candidateFloor + 0.04;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            epsStepTarget := Max(candidateFloor, Min(candidateCeil, epsStepTarget));

            adaptAlpha := 0.38;
            if auditProtActive then
              adaptAlpha := 0.20
            else if (volatilityState = 2) then
              adaptAlpha := 0.28
            else if (Abs(epsStepTarget - epsStep) > 0.10) then
              adaptAlpha := 0.48;

            if (epsStep <= 0) then
              epsStep := epsStepTarget
            else
              epsStep := epsStep + adaptAlpha * (epsStepTarget - epsStep);

            epsStep := Max(candidateFloor, Min(candidateCeil, epsStep));

            epsRetTarget := 0.10 + (volatilityState * 0.025) + Max(0.0, regimeAdjExit * 0.16);
            if (regimeMercado = 3) then
              epsRetTarget := epsRetTarget - 0.03
            else if (regimeMercado = 2) then
              epsRetTarget := epsRetTarget + 0.03;
            if (bodyATR >= 0.35) then
              epsRetTarget := epsRetTarget - 0.02;
            if (barsDesdeSinal >= 6) then
              epsRetTarget := epsRetTarget - Min(0.03, (barsDesdeSinal - 5) * 0.006);
            candidateFloor := 0.07 + (volatilityState * 0.015);
            candidateCeil  := 0.34 + Max(0.0, regimeAdjExit * 0.18);
            epsRet := Max(candidateFloor, Min(candidateCeil, epsRetTarget));

            if (not optimalExit) and (atr0 > 0) then
            begin
              pnlATR  := (entryPrice - lowSinceEntry) / Max(0.0001, atr0);
              moveATR := (Max(Close[0], currentHigh) - lowSinceEntry) / Max(0.0001, atr0);

              if (pnlATR >= epsStep) and
                 (
                   // Caminho clássico: já houve repique/micro-reversão suficiente.
                   (moveATR >= epsRet) or

                   // Caminho aprimorado: exaustão executável por pavio inferior + VRS.
                   (
                     (lowerWickCurrent >= ratioThresh) and
                     (vrs0 >= Max(0.0001, vrsMedia0) * (0.92 + (volatilityState * 0.04))) and
                     (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.80))
                   )
                 ) then
              begin
                optimalExit    := True;
                exitByReversal := True;

                if (
                     (lowerWickCurrent >= ratioThresh) and
                     (vrs0 >= Max(0.0001, vrsMedia0) * (0.92 + (volatilityState * 0.04))) and
                     (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.80))
                   ) then
                begin
                  forceExit5m := True;

                  if (xaiExitCause = "") then
                    xaiExitCause := "EXIT=EXHAUST_WICK_VRS_BOTTOM_SELL";
                end
                else
                begin
                  if (xaiExitCause = "") then
                    xaiExitCause := "EXIT=EXTREME_MICROREV_SELL";
                end;
              end;
            end;

            // =====================================================
            // SCORE XAI COMO FALLBACK REAL
            // =====================================================
            confidenceScore := 0;
            if condition1 then confidenceScore := confidenceScore + 25;
            if condition2 then confidenceScore := confidenceScore + 30;
            if condition3 then confidenceScore := confidenceScore + 20;
            if condition4 then confidenceScore := confidenceScore + 20;
            if condition5 then confidenceScore := confidenceScore + 15;

            confidenceThreshold := 46 + (volatilityState * 5) + Max(0.0, regimeAdjExit * 8);

            if (regimeMercado = 0) then
              confidenceThreshold := confidenceThreshold + 4
            else if (regimeMercado = 2) then
              confidenceThreshold := confidenceThreshold + 2
            else if (regimeMercado = 3) then
              confidenceThreshold := confidenceThreshold - 3;

            confidenceThreshold := confidenceThreshold - Min(14, barsDesdeSinal * 1.4);

            if (maxGainATR >= minGainExitATR) then
              confidenceThreshold := confidenceThreshold - 6;

            if (auditExpectancy_ATR < 0) then
              confidenceThreshold := confidenceThreshold + Min(8, Abs(auditExpectancy_ATR) * 6)
            else
              confidenceThreshold := confidenceThreshold - Min(4, auditExpectancy_ATR * 4);

            if (auditProfitFactor < 1.0) then
              confidenceThreshold := confidenceThreshold + Min(6, (1.0 - auditProfitFactor) * 10)
            else
              confidenceThreshold := confidenceThreshold - Min(3, (auditProfitFactor - 1.0) * 4);

            if (auditEwmaWin < 0.50) then
              confidenceThreshold := confidenceThreshold + Min(6, (0.50 - auditEwmaWin) * 18)
            else
              confidenceThreshold := confidenceThreshold - Min(3, (auditEwmaWin - 0.50) * 10);

            candidateFloor := 24 + (volatilityState * 2) + Max(0.0, regimeAdjExit * 6);
            candidateCeil  := 82 - Min(6, regimeStrength / 18.0);

            if auditProtActive then
              candidateFloor := candidateFloor + 4;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            confidenceThreshold := Max(candidateFloor, Min(candidateCeil, confidenceThreshold));

            // =====================================================
            confidenceAmarelo := (confidenceScore >= confidenceThreshold);

            if (not optimalExit) then
            begin
              pnlATR := (entryPrice - lowSinceEntry) / Max(0.0001, atr0);

              if confidenceAmarelo and
                 (pnlATR >= Max(minGainExitATR * 0.76, epsStep * 0.74)) and
                 (condition3 or (offsetMinima <= 1)) and
                 (moveATR >= (epsRet * 0.45)) then
              begin
                optimalExit := True;
                if (xaiExitCause = "") then xaiExitCause := "EXIT=CONFIRM_XAI_AFTER_EXTREME_SELL";
              end;
            end;

            // PROTEÇÃO CONTRA AMARELO PRECOCE EM VOL ALTA
            // Correção-Σ:
            // - continua filtrando ruído verdadeiro;
            // - mas NÃO revoga um AMARELO já validado quando já existe
            //   reversão executável + lucro líquido mínimo acima do piso de custo;
            // - em atraso > 1 barra após o fundo, a prioridade passa a ser
            //   travar lucro, não insistir em permanência.
            // =====================================================
            if (volatilityState = 2) and (optimalExit) and (not forceExit5m) and (not exitByStop) and
              (offsetMinima > 1) then
            begin
              pnlATR  := (entryPrice - lowSinceEntry) / Max(0.0001, atr0);
              moveATR := (Max(Close[0], currentHigh) - lowSinceEntry) / Max(0.0001, atr0);
              candidateFloor := Max(0.22, Min(0.48, minGainExitATR * 0.78));

              // Se já há lucro líquido mínimo e reversão causal executável,
              // o AMARELO deve sobreviver.
              if (pnlATR >= costFloorATR) and
                 (moveATR >= Max(0.05, epsRet * 0.40)) and
                 (
                   (Close[0] > Close[1]) or
                   (currentHigh >= High[1]) or
                   (Close[0] > Open[0])
                 ) then
              begin
                optimalExit := True;

                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=POST_BOTTOM_CONFIRM_EXEC";
              end

              else if (xaiExitCause <> "EXIT=EXTREME_MICROREV_SELL") and
                      (xaiExitCause <> "EXIT=SELL_REVERSAL_LOW_RISK") and
                      (xaiExitCause <> "EXIT=SELL_REVERSAL_EDGE_EXEC") and
                      (xaiExitCause <> "EXIT=STRUCT_REV_AFTER_BOTTOM_EXEC") and
                      (xaiExitCause <> "EXIT=POST_BOTTOM_CONFIRM_EXEC") and
                      (xaiExitCause <> "EXIT=CONFIRM_XAI_AFTER_EXTREME_SELL") and
                      (xaiExitCause <> "EXIT=TIME_DECAY_PROFIT_LOCK_SELL") and
                      (xaiExitCause <> "EXIT=EXHAUST_WICK(+BB)+PROFIT_OK_SELL") then
              begin
                if (bodyATR < 0.12) and
                   (moveATR < (epsRet * 0.72)) and
                   (pnlATR < Max(candidateFloor, epsStep + 0.08)) then
                begin
                  optimalExit := False;
                  xaiExitCause := "BLOCK:VOL_NOISE_WEAKREV_SELL";
                end;

                if optimalExit and
                   ((Min(Open[0], Close[0]) - currentLow) > (0.72 * currentRange)) and
                   (moveATR < (epsRet * 0.92)) and
                   (pnlATR < Max(candidateFloor, minGainExitATR * 0.88)) then
                begin
                  optimalExit := False;
                  xaiExitCause := "BLOCK:VOL_SPIKE_CONT_SELL";
                end;

                if optimalExit and condition3 and
                   (Close[0] > (bbBot[0] + 0.08 * atr0)) and
                   (moveATR < (epsRet * 0.92)) and
                   (pnlATR < Max(candidateFloor, minGainExitATR * 0.88)) then
                begin
                  optimalExit := False;
                  xaiExitCause := "BLOCK:VOL_LEVELTEST_WEAKREV_SELL";
                end;
              end;

              if (not optimalExit) then
              begin
                // Correção-Σ:
                // Não reposicionar stop usando informação intrabar já conhecida.
                // Isso evita recompra/stop falso no mesmo candle.
                // A proteção real continua soberana no ProfitGuardian/trailing já calculado.
                if (xaiGuardNote = "") then
                  xaiGuardNote := "PG=VOL_HIGH_NO_RETRO_STOP_SELL";
              end;
            end;     
          end;

          // V. FALLBACK SOBERANO DO PONTO ÓTIMO — SEM ESPERAR SINAL OPOSTO
          // CORREÇÃO-Σ v2:
          // O atraso crítico estava em limitar a saída útil a offset <= 1
          // e depois esperar MaxBarrasTrade/2 para o time-decay.
          // Agora o AMARELO nasce na primeira barra executável com:
          // topo/fundo já conhecido + devolução material + lucro líquido preservado.
          // Não exige sinal oposto completo, não usa função nova e não cria estado duplicado.

          if (not optimalExit) then
          begin
            // lucro mínimo adaptativo científico — supervisionado por regime/auditoria
            minProfitATR := minGainExitATR;

            candidateFloor := 0.18 + (volatilityState * 0.025);
            candidateCeil  := 0.56 + Min(0.10, regimeStrength / 420.0);

            if (regimeMercado = 3) then
              candidateFloor := Max(0.15, candidateFloor - 0.025);

            if (regimeMercado = 0) then
              candidateCeil := Min(0.70, candidateCeil + 0.04);

            if auditProtActive then
              candidateFloor := candidateFloor + 0.02;

            if (candidateCeil < candidateFloor) then
              candidateCeil := candidateFloor;

            minProfitATR := Max(candidateFloor, Min(candidateCeil, minProfitATR));

            // ================= BUY =================
            if holdBuy then
            begin
              offsetMaxima := CurrentBar() - hiBarIdx;
              if (offsetMaxima < 0) then
                offsetMaxima := 0;

              pnlATR           := (highSinceEntry - entryPrice) / Max(0.0001, atr0);
              moveATR          := (highSinceEntry - Min(Close[0], currentLow)) / Max(0.0001, atr0);
              currentProfitATR := (Close[0] - entryPrice) / Max(0.0001, atr0);

              // Janela dinâmica curta — refinamento cirúrgico anti-atraso.
              // Em vol alta, não alonga espera se já existe lucro líquido executável.
              lagIdx := 1;

              if (volatilityState = 1) then
                lagIdx := 2
              else if (volatilityState = 2) then
              begin
                if (pnlATR >= Max(costFloorATR + Capture_MinNetATR, minProfitATR * 0.70)) then
                  lagIdx := 1
                else
                  lagIdx := 2;
              end;

              if (regimeMercado = 3) then
                lagIdx := 1;

              if (barsDesdeSinal >= 6) then
                lagIdx := Max(1, lagIdx - 1);

              // condition1 = ainda está dentro da janela executável do topo.
              condition1 := (CurrentBar() >= hiBarIdx) and (offsetMaxima <= lagIdx);

              // condition2 = reversão/devolução causal; sem esperar venda completa.
              condition2 := (Close[0] < Open[0]) or
                            (upperWickCurrent > (ratioThresh * 0.70)) or
                            (moveATR >= Max(0.040, epsRet * 0.16));

              if (CurrentBar() > 0) then
                condition2 := condition2 or
                              (Close[0] < Close[1]) or
                              (Low[0] < Low[1]);

              // condition3 = ainda existe lucro líquido executável no fechamento da barra.
              condition3 := (pnlATR >= Max(costFloorATR + Capture_MinNetATR, minProfitATR * 0.55)) and
                            (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.80));

              if condition1 and condition2 and condition3 then
              begin
                forceExit5m    := True;
                optimalExit    := True;
                exitByReversal := True;

                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=BUY_FIRST_EXEC_REV_NO_OPPOSITE_WAIT";
              end

              // Segunda trava: se passou da janela curta, não espera MaxBarrasTrade/2.
              // Protege lucro quando a devolução já é material e o fechamento ainda paga custo.
              else if (offsetMaxima > lagIdx) and
                      (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.60)) and
                      (pnlATR >= Max(costFloorATR + Capture_MinNetATR, minProfitATR * 0.50)) and
                      (moveATR >= Max(0.070, epsRet * 0.25)) then
              begin
                forceExit5m    := True;
                optimalExit    := True;
                exitByReversal := True;
                xaiExitCause   := "EXIT=TIME_DECAY_PROFIT_LOCK_BUY";
              end

              else
              begin
                if (xaiExitCause = "") then
                  xaiExitCause := "BLOCK:BUY_WAIT_EXEC_REV";
              end;
            end;

            // ================= SELL =================
            if holdSell then
            begin
              offsetMinima := CurrentBar() - loBarIdx;
              if (offsetMinima < 0) then
                offsetMinima := 0;

              pnlATR           := (entryPrice - lowSinceEntry) / Max(0.0001, atr0);
              moveATR          := (Max(Close[0], currentHigh) - lowSinceEntry) / Max(0.0001, atr0);
              currentProfitATR := (entryPrice - Close[0]) / Max(0.0001, atr0);

              // Janela dinâmica curta: lateral sai mais rápido; vol alta permite 1 candle extra.
              lagIdx := 1 + volatilityState;

              if (regimeMercado = 3) then
                lagIdx := 1;

              if (barsDesdeSinal >= 6) then
                lagIdx := Max(1, lagIdx - 1);

              // condition1 = ainda está dentro da janela executável do fundo.
              condition1 := (CurrentBar() >= loBarIdx) and (offsetMinima <= lagIdx);

              // condition2 = repique/devolução causal; sem esperar compra completa.
              condition2 := (Close[0] > Open[0]) or
                            (lowerWickCurrent > (ratioThresh * 0.70)) or
                            (moveATR >= Max(0.040, epsRet * 0.16));

              if (CurrentBar() > 0) then
                condition2 := condition2 or
                              (Close[0] > Close[1]) or
                              (High[0] > High[1]);

              // condition3 = ainda existe lucro líquido executável no fechamento da barra.
              condition3 := (pnlATR >= Max(costFloorATR + Capture_MinNetATR, minProfitATR * 0.55)) and
                            (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.80));

              if condition1 and condition2 and condition3 then
              begin
                forceExit5m    := True;
                optimalExit    := True;
                exitByReversal := True;

                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=SELL_FIRST_EXEC_REV_NO_OPPOSITE_WAIT";
              end

              // Segunda trava: se passou da janela curta, não espera MaxBarrasTrade/2.
              // Protege lucro quando a devolução já é material e o fechamento ainda paga custo.
              else if (offsetMinima > lagIdx) and
                      (currentProfitATR >= Max(costFloorATR, Capture_MinNetATR * 0.60)) and
                      (pnlATR >= Max(costFloorATR + Capture_MinNetATR, minProfitATR * 0.50)) and
                      (moveATR >= Max(0.070, epsRet * 0.25)) then
              begin
                forceExit5m    := True;
                optimalExit    := True;
                exitByReversal := True;
                xaiExitCause   := "EXIT=TIME_DECAY_PROFIT_LOCK_SELL";
              end

              else
              begin
                if (xaiExitCause = "") then
                  xaiExitCause := "BLOCK:SELL_WAIT_EXEC_REV";
              end;
            end;
          end;

          // VI. STOP LOSS ADAPTATIVO SOBERANO
          // Regra pétrea:
          // 1) a execução real do stop usa EXCLUSIVAMENTE o slPrice dinâmico;
          // 2) toque intrabar no stop é SOBERANO e arma a saída no próprio candle;
          // 3) o AMARELO continua sendo pintado SOMENTE no bloco canônico de aplicação final,
          //    evitando dupla autoridade sobre paintStateSeries[0].
          stopTouchedBar := False;
          stopTouchPx    := 0;

          if holdBuy then
          begin
            slCandidate := slPrice;

            if (slCandidate <= 0) then
            begin
              auditProtActive   := True;
              auditProtBarsLeft := Max(auditProtBarsLeft, 1);
              gateOpen          := False;
              allowTrade        := False;
              adaptiveGateOk    := False;
              xaiBlockReason    := "FALHA_VALIDACAO:SLPRICE_INVALIDO_BUY";
            end
            else if (currentLow <= slCandidate) then
            begin
              // STOP ATÔMICO:
              // Tocou o stop válido da compra -> saída armada neste mesmo candle.
              stopTouchedBar  := True;
              stopTouchPx     := slCandidate;
              optimalExit     := True;
              exitByStop      := True;
              exitByReversal  := False;
              exitByConfidence := False;
              forceExit5m     := True;
              xaiExitCause    := "EXIT=STOP(SLPRICE_REAL_BUY)";
            end;
          end;

          if holdSell then
          begin
            slCandidate := slPrice;

            if (slCandidate <= 0) then
            begin
              auditProtActive   := True;
              auditProtBarsLeft := Max(auditProtBarsLeft, 1);
              gateOpen          := False;
              allowTrade        := False;
              adaptiveGateOk    := False;
              xaiBlockReason    := "FALHA_VALIDACAO:SLPRICE_INVALIDO_SELL";
            end
            else if (currentHigh >= slCandidate) then
            begin
              // STOP ATÔMICO:
              // Tocou o stop válido da venda -> saída armada neste mesmo candle.
              stopTouchedBar  := True;
              stopTouchPx     := slCandidate;
              optimalExit     := True;
              exitByStop      := True;
              exitByReversal  := False;
              exitByConfidence := False;
              forceExit5m     := True;
              xaiExitCause    := "EXIT=STOP(SLPRICE_REAL_SELL)";
            end;
          end;

          // XAI: se saiu pela confirmação (confidenceAmarelo), registra explicitamente
          if (confidenceAmarelo) and (optimalExit) and (xaiExitCause = "") then
          begin
            exitByConfidence := True;
            xaiExitCause     := "EXIT=CONFIRM(XAI_SCORE)";
          end;
          
          // 4) APLICAÇÃO DO ESTADO FINAL
          if holdBuy then
          begin
            if optimalExit then
            begin
              // SAÍDA NA COMPRA — EXECUÇÃO REAL SEM REPAINT
              // Regra pétrea:
              // 1) O AMARELO SEMPRE nasce no candle atual (executável).
              // 2) O topo ótimo real continua existindo, mas só para auditoria.
              // 3) Nunca pintar candle passado como se a execução tivesse ocorrido lá.

              paintStateSeries[0] := 2;   // AMARELO EXECUTÁVEL = barra atual

              if exitByStop then
              begin
                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=STOP_REAL_BUY";

                slCandidate := stopTouchPx;
                if (slCandidate <= 0) then
                  slCandidate := slPrice;

                if (slCandidate <= 0) then
                begin
                  auditProtActive   := True;
                  auditProtBarsLeft := Max(auditProtBarsLeft, 1);
                  xaiBlockReason    := "FALHA_VALIDACAO:SLPRICE_INVALIDO_BUY_AUDIT";
                  auditExitPx       := Close[0];
                end
                else
                begin
                  auditExitPx := slCandidate;

                  // Gap adverso: abriu abaixo do stop -> fill realista na abertura, não no stop nominal.
                  if (Open[0] < auditExitPx) then
                    auditExitPx := Open[0];
                end;

                auditExitBar := CurrentBar();
              end

              else
              begin
                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=PO_MAX_EXECUTAVEL";

                // Auditoria separa "ponto ótimo teórico" de "preço executável real"
                auditExitPx := Close[0];
                auditExitBar := CurrentBar();
              end;

              if ModoDebug then
                PlotText("SAIR AGORA (PO_MAX=EXEC_REAL)", clYellow, 2, 7);          

              // ==============================
              // TRADE AUDITOR — BUY (verde → amarelo) — CORRIGIDO
              // ==============================
              auditEntryPx     := entryPrice;
              auditMFE         := Max(0.0001, highSinceEntry - auditEntryPx);
              auditMAE         := Max(0.0,     auditEntryPx - lowSinceEntry);

              auditPnLBruto := (auditExitPx - auditEntryPx);

              qtyExitEff := Max(1.0, Abs(PositionQty));

              if Exec_UsarCustosReais then
                execCostTicks := execCostTicksEff
              else
                execCostTicks := 0.0;

              execCostPx   := execCostTicks * MinPriceIncrement * qtyExitEff;
              auditPnL     := (auditPnLBruto * qtyExitEff) - execCostPx;
              auditPnL_ATR := (auditPnL / Max(1.0, qtyExitEff)) / Max(0.0001, atr0);

              // ===== MÉTRICA DE CAPTURA DO AMARELO — COMPRA =====
              // extremo teórico = máxima desde a entrada; saída real estimada = preço auditado.
              yellowExtremePx     := highSinceEntry;
              yellowTheoreticalPx := yellowExtremePx;

              // COMPRA:
              // O extremo teórico continua sendo High[0]/highSinceEntry.
              // Porém o AMARELO executável deve ser auditado pelo preço real de saída.
              // Isso separa "ponto ótimo teórico" de "preço possível de execução".
              yellowVisualPx      := auditExitPx;

              yellowRealPx        := auditExitPx;
              yellowNetATR        := auditPnL_ATR;
              yellowGivebackATR   := Max(0.0, (yellowExtremePx - yellowRealPx) / Max(0.0001, atr0));
              yellowFillDriftTicks := Abs(yellowVisualPx - yellowRealPx) / Max(0.0001, MinPriceIncrement);

              yellowCaptureEff := (yellowRealPx - auditEntryPx) / Max(0.0001, yellowExtremePx - auditEntryPx);
              if yellowCaptureEff < 0 then yellowCaptureEff := 0;
              if yellowCaptureEff > 1 then yellowCaptureEff := 1;

              yellowCaptureOk := (yellowNetATR >= Capture_MinNetATR) and
                                 (yellowCaptureEff >= Capture_MinEfficiency) and
                                 (yellowGivebackATR <= Capture_MaxGiveBack_ATR) and
                                 (yellowFillDriftTicks <= Exec_MaxFillDrift_Ticks);

              // ===== GESTÃO DIÁRIA VIVA (pós-trade) =====
              dRes := dRes + auditPnL;
              if (dRes > dResPeak) then dResPeak := dRes;

              if (auditPnL <= 0) then
              begin
                lossesSeguidas := lossesSeguidas + 1;
                cooling := CooldownLoss_Bars;
              end
              else
                lossesSeguidas := 0;

              if (StopDia_Loss > 0) and (dRes <= -StopDia_Loss) then
              begin
                travaDia        := True;
                auditProtReason := "STOP_DIA_LOSS";
              end;

              if (StopDia_Gain > 0) and (dRes >= StopDia_Gain) then
              begin
                travaGanho      := True;
                auditProtReason := "STOP_DIA_GAIN";
              end;

              if (Kill_ConsecLoss > 0) and (lossesSeguidas >= Kill_ConsecLoss) then
              begin
                travaDia        := True;
                auditProtReason := "KILL_CONSECLOSS";
              end;

              auditEfficiency  := auditPnL / auditMFE;
              if auditEfficiency < 0 then auditEfficiency := 0;
              if auditEfficiency > 1 then auditEfficiency := 1;

              auditExitReason  := xaiExitCause;
              auditEntryReason := xaiSignalReason;

              // --- Normalização por regime (institucional) ---
              regimeAdjExit := 0.55 + (volatilityState * 0.06);
              if (regimeMercado = 2) then regimeAdjExit := regimeAdjExit + 0.10;
              if (regimeMercado = 1) then regimeAdjExit := regimeAdjExit - 0.04;
              if (regimeAdjExit < 0.45) then regimeAdjExit := 0.45;
              if (regimeAdjExit > 0.80) then regimeAdjExit := 0.80;

              entropySignal := 0.30 + (volatilityState * 0.08);
              if (regimeMercado = 2) then entropySignal := entropySignal + 0.04;
              if (entropySignal < 0.22) then entropySignal := 0.22;
              if (entropySignal > 0.55) then entropySignal := 0.55;

              auditEffNorm  := auditEfficiency / Max(0.0001, regimeAdjExit);
              if auditEffNorm < 0 then auditEffNorm := 0;
              if auditEffNorm > 1 then auditEffNorm := 1;

              auditPnLNorm  := auditPnL_ATR / Max(0.0001, entropySignal);
              if auditPnLNorm < 0 then auditPnLNorm := 0;
              if auditPnLNorm > 1 then auditPnLNorm := 1;

              auditPainNorm := 1 - Min(1, auditMAE / Max(0.0001, auditMFE));
              if auditPainNorm < 0 then auditPainNorm := 0;
              if auditPainNorm > 1 then auditPainNorm := 1;

              confHi := 0.55;  // peso eficiência
              confLo := 0.35;  // peso pnl

              // peso implícito da dor = 1 - confHi - confLo
              if (regimeMercado = 3) then
              begin
                confHi := 0.45;
                confLo := 0.30;
              end
              else if (regimeMercado = 0) then
              begin
                confHi := 0.60;
                confLo := 0.30;
              end
              else if (volatilityState = 2) then
              begin
                confHi := 0.50;
                confLo := 0.25;
              end;

              auditScore := Round(100 * (
                             (confHi * auditEffNorm) +
                             (confLo * auditPnLNorm) +
                             ((1.0 - confHi - confLo) * auditPainNorm)
                           ));
              if auditScore < 0 then auditScore := 0;
              if auditScore > 100 then auditScore := 100;

              // Correção-Σ: a captura ruim do AMARELO precisa punir COMPRA e VENDA de forma simétrica.
              // Sem isso, uma saída comprada que devolve lucro demais pode receber nota artificialmente alta.
              if (not yellowCaptureOk) then
                auditScore := Max(0, auditScore - Round(10 + (yellowGivebackATR * 10)));

              if (auditPnL_ATR < 0.0) then auditTag := "RUIM"
              else if (not yellowCaptureOk) then auditTag := "C_CAPTURA"
              else if (auditScore >= 85) then auditTag := "A"
              else if (auditScore >= 70) then auditTag := "B"
              else auditTag := "C";

              // ==============================
              // WALK-FORWARD / FORA DA AMOSTRA
              // ==============================
              if WF_Ativo then
              begin
                // Classifica o trade fechado pela BARRA DE SAÍDA, não pela barra de entrada.
                // Isso evita vazamento de informação quando um trade nasce no treino e fecha na OOS.
                if (((auditExitBar - (Floor(auditExitBar / wfCycleBars) * wfCycleBars)) < WF_BarrasTreino) and
                    (Floor(auditExitBar / wfCycleBars) = wfFoldId)) then
                begin
                  wfTrainTrades := wfTrainTrades + 1;
                  wfTrainEquity_ATR := wfTrainEquity_ATR + auditPnL_ATR;
                  if (wfTrainEquity_ATR > wfTrainPeak_ATR) then
                    wfTrainPeak_ATR := wfTrainEquity_ATR;

                  wfTrainDD_ATR := wfTrainPeak_ATR - wfTrainEquity_ATR;
                  if (wfTrainDD_ATR > wfTrainMaxDD_ATR) then
                    wfTrainMaxDD_ATR := wfTrainDD_ATR;

                  if (auditPnL_ATR > 0) then
                  begin
                    wfTrainWins := wfTrainWins + 1.0;
                    wfTrainGain_ATR := wfTrainGain_ATR + auditPnL_ATR;
                  end
                  else
                    wfTrainLoss_ATR := wfTrainLoss_ATR + Abs(auditPnL_ATR);

                  wfTrainWinRate := wfTrainWins / Max(1, wfTrainTrades);
                  wfTrainExp_ATR := wfTrainEquity_ATR / Max(1, wfTrainTrades);
                  wfTrainPF      := wfTrainGain_ATR / Max(0.0001, wfTrainLoss_ATR);
                end
                else
                begin
                  if (Floor(entryBarIdx / wfCycleBars) <> Floor(auditExitBar / wfCycleBars)) and (wfBlockReason = "") then
                    wfBlockReason := "WF=CARRY_TRADE";

                  wfOOSTrades := wfOOSTrades + 1;
                  wfOOSEquity_ATR := wfOOSEquity_ATR + auditPnL_ATR;
                  if (wfOOSEquity_ATR > wfOOSPeak_ATR) then
                    wfOOSPeak_ATR := wfOOSEquity_ATR;

                  wfOOSDD_ATR := wfOOSPeak_ATR - wfOOSEquity_ATR;
                  if (wfOOSDD_ATR > wfOOSMaxDD_ATR) then
                    wfOOSMaxDD_ATR := wfOOSDD_ATR;

                  if (auditPnL_ATR > 0) then
                  begin
                    wfOOSWins := wfOOSWins + 1.0;
                    wfOOSGain_ATR := wfOOSGain_ATR + auditPnL_ATR;
                  end
                  else
                    wfOOSLoss_ATR := wfOOSLoss_ATR + Abs(auditPnL_ATR);

                  wfOOSWinRate := wfOOSWins / Max(1, wfOOSTrades);
                  wfOOSExp_ATR := wfOOSEquity_ATR / Max(1, wfOOSTrades);
                  wfOOSPF      := wfOOSGain_ATR / Max(0.0001, wfOOSLoss_ATR);
                end;
              end;

              // ==============================
              // MÉTRICAS ADAPTATIVAS (EWMA) + PROTEÇÃO
              // ==============================
              if (auditTrades = 0) then
              begin
                if (auditPnL_ATR > 0) then
                  auditEwmaWin := 1.0
                else
                  auditEwmaWin := 0.0;

                if (auditPnL_ATR > 0) then
                  auditEwmaGain_ATR := auditPnL_ATR
                else
                  auditEwmaGain_ATR := 0.0;

                if (auditPnL_ATR < 0) then
                  auditEwmaLoss_ATR := Abs(auditPnL_ATR)
                else
                  auditEwmaLoss_ATR := 0.0;

                auditEwmaEff := auditEfficiency;

                // Correção-Σ:
                // Não carregar auditPnL_ATR aqui.
                // A soma oficial acontece uma única vez no bloco:
                // auditEquity_ATR := auditEquity_ATR + auditPnL_ATR;
                auditEquity_ATR := 0.0;
                auditPeak_ATR   := 0.0;
                auditDD_ATR     := 0.0;
                auditMaxDD_ATR  := 0.0;

                auditProtBarsLeft := 0;
                auditProtReason   := "";
              end;

              auditTrades := auditTrades + 1;

              auditAlpha := 0.08 + (volatilityState * 0.04) + (Abs(marketMomentum) * 0.03);
              if auditAlpha < 0.05 then auditAlpha := 0.05;
              if auditAlpha > 0.25 then auditAlpha := 0.25;

              if (auditPnL_ATR > 0) then
                auditEwmaWin := ((1 - auditAlpha) * auditEwmaWin) + (auditAlpha * 1)
              else
                auditEwmaWin := ((1 - auditAlpha) * auditEwmaWin);

              if (auditPnL_ATR > 0) then
                auditEwmaGain_ATR := ((1 - auditAlpha) * auditEwmaGain_ATR) + (auditAlpha * auditPnL_ATR)
              else
                auditEwmaGain_ATR := ((1 - auditAlpha) * auditEwmaGain_ATR);

              if (auditPnL_ATR < 0) then
                auditEwmaLoss_ATR := ((1 - auditAlpha) * auditEwmaLoss_ATR) + (auditAlpha * Abs(auditPnL_ATR))
              else
                auditEwmaLoss_ATR := ((1 - auditAlpha) * auditEwmaLoss_ATR);

              auditEwmaEff := ((1 - auditAlpha) * auditEwmaEff) + (auditAlpha * auditEfficiency);

              auditExpectancy_ATR := auditEwmaGain_ATR - auditEwmaLoss_ATR;
              auditProfitFactor   := auditEwmaGain_ATR / Max(0.0001, auditEwmaLoss_ATR);

              auditEquity_ATR := auditEquity_ATR + auditPnL_ATR;
              if (auditEquity_ATR > auditPeak_ATR) then auditPeak_ATR := auditEquity_ATR;
              auditDD_ATR := auditPeak_ATR - auditEquity_ATR;
              if (auditDD_ATR > auditMaxDD_ATR) then auditMaxDD_ATR := auditDD_ATR;

              if (auditTrades >= 8) then
              begin
                regimeAdjExit := 0.42 - (volatilityState * 0.04);
                if (regimeAdjExit < 0.28) then regimeAdjExit := 0.28;
                if (regimeAdjExit > 0.46) then regimeAdjExit := 0.46;

                entropySignal := 2.20 + (volatilityState * 0.60);
                if (entropySignal < 1.80) then entropySignal := 1.80;
                if (entropySignal > 3.80) then entropySignal := 3.80;

                if (auditDD_ATR > entropySignal) or ((auditExpectancy_ATR < 0) and (auditEwmaWin < regimeAdjExit) and (auditProfitFactor < 1.0)) then
                begin
                  auditProtBarsLeft := Round(3 + (2 * volatilityState));
                  if auditProtBarsLeft < 3 then auditProtBarsLeft := 3;
                  if auditProtBarsLeft > 9 then auditProtBarsLeft := 9;

                  if (auditDD_ATR > entropySignal) then
                    auditProtReason := "PROTECT:DD_HIGH"
                  else
                    auditProtReason := "PROTECT:EXPECT_NEG";

                  if (auditTag = "RUIM") and (auditProtBarsLeft < (4 + volatilityState)) then
                    auditProtBarsLeft := (4 + volatilityState);
                end
                else
                begin
                  if (auditProtBarsLeft = 0) then auditProtReason := "";
                end;
              end;

              // ==============================
              // RESET DO CICLO
              // ==============================
              lastEntrySideExit := lastEntrySide;
 
              if SomenteSinais then
              begin
                holdBuy        := False;
                lastEntrySide  := 0;

                // RESET PÉTREO PÓS-AMARELO:
                // Este bloco só executa dentro de optimalExit, portanto o candle atual já é AMARELO.
                // O próximo candle deve ser NEUTRO obrigatório.
                resetBars := 1;
              end
 
              else
              begin
                exitPending     := True;
                exitPendingSide := 1;
                exitSignalBar   := CurrentBar();
                resetBars       := 0;
              end;
            end

            else
            begin
              // PERSISTÊNCIA DO VERDE (trade continua ativo, não saiu neste candle)
              paintStateSeries[0] := 1;
            end;
          end
          else if holdSell then
          begin
             if optimalExit then
            begin
              // SAÍDA NA VENDA — EXECUÇÃO REAL SEM REPAINT
              // Regra pétrea:
              // 1) O AMARELO SEMPRE nasce no candle atual (executável).
              // 2) O fundo ótimo real fica só para auditoria.
              // 3) Nunca recolorir barra passada como se a recompra tivesse acontecido lá.

              paintStateSeries[0] := 2;   // AMARELO EXECUTÁVEL = barra atual

              if exitByStop then
              begin
                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=STOP_REAL_SELL";

                slCandidate := stopTouchPx;
                if (slCandidate <= 0) then
                  slCandidate := slPrice;

                if (slCandidate <= 0) then
                begin
                  auditProtActive   := True;
                  auditProtBarsLeft := Max(auditProtBarsLeft, 1);
                  xaiBlockReason    := "FALHA_VALIDACAO:SLPRICE_INVALIDO_SELL_AUDIT";
                  auditExitPx       := Close[0];
                end
                else
                begin
                  auditExitPx := slCandidate;

                  // Gap adverso: abriu acima do stop -> fill realista na abertura, não no stop nominal.
                  if (Open[0] > auditExitPx) then
                    auditExitPx := Open[0];
                end;

                auditExitBar := CurrentBar();
              end

              else
              begin
                if (xaiExitCause = "") then
                  xaiExitCause := "EXIT=PO_MIN_EXECUTAVEL";

                auditExitPx := Close[0];
                auditExitBar := CurrentBar();
              end;

              if ModoDebug then
                PlotText("SAIR AGORA (PO_MIN=EXEC_REAL)", clYellow, 2, 7); 

              // ==============================
              // TRADE AUDITOR — SELL (vermelho → amarelo) — CORRIGIDO
              // ==============================
              auditEntryPx     := entryPrice;
              auditMFE         := Max(0.0001, auditEntryPx - lowSinceEntry);
              auditMAE         := Max(0.0,     highSinceEntry - auditEntryPx);

              auditPnLBruto := (auditEntryPx - auditExitPx);

              qtyExitEff := Max(1.0, Abs(PositionQty));

              if Exec_UsarCustosReais then
                execCostTicks := execCostTicksEff
              else
                execCostTicks := 0.0;

              execCostPx   := execCostTicks * MinPriceIncrement * qtyExitEff;
              auditPnL     := (auditPnLBruto * qtyExitEff) - execCostPx;
              auditPnL_ATR := (auditPnL / Max(1.0, qtyExitEff)) / Max(0.0001, atr0);

              // ===== MÉTRICA DE CAPTURA DO AMARELO — VENDA =====
              // extremo teórico = mínima desde a entrada; saída real estimada = preço auditado.
              yellowExtremePx     := lowSinceEntry;
              yellowTheoreticalPx := yellowExtremePx;

              // VENDA:
              // O extremo teórico continua sendo Low[0]/lowSinceEntry.
              // Porém o AMARELO executável deve ser auditado pelo preço real de saída.
              // Isso separa "ponto ótimo teórico" de "preço possível de execução".
              yellowVisualPx      := auditExitPx;

              yellowRealPx        := auditExitPx;
              yellowNetATR        := auditPnL_ATR;
              yellowGivebackATR   := Max(0.0, (yellowRealPx - yellowExtremePx) / Max(0.0001, atr0));
              yellowFillDriftTicks := Abs(yellowVisualPx - yellowRealPx) / Max(0.0001, MinPriceIncrement);

              yellowCaptureEff := (auditEntryPx - yellowRealPx) / Max(0.0001, auditEntryPx - yellowExtremePx);
              if yellowCaptureEff < 0 then yellowCaptureEff := 0;
              if yellowCaptureEff > 1 then yellowCaptureEff := 1;

              yellowCaptureOk := (yellowNetATR >= Capture_MinNetATR) and
                                 (yellowCaptureEff >= Capture_MinEfficiency) and
                                 (yellowGivebackATR <= Capture_MaxGiveBack_ATR) and
                                 (yellowFillDriftTicks <= Exec_MaxFillDrift_Ticks);

              // ===== GESTÃO DIÁRIA VIVA (pós-trade) =====
              dRes := dRes + auditPnL;
              if (dRes > dResPeak) then dResPeak := dRes;

              if (auditPnL <= 0) then
              begin
                lossesSeguidas := lossesSeguidas + 1;
                cooling := CooldownLoss_Bars;
              end
              else
                lossesSeguidas := 0;

              if (StopDia_Loss > 0) and (dRes <= -StopDia_Loss) then
              begin
                travaDia        := True;
                auditProtReason := "STOP_DIA_LOSS";
              end;

              if (StopDia_Gain > 0) and (dRes >= StopDia_Gain) then
              begin
                travaGanho      := True;
                auditProtReason := "STOP_DIA_GAIN";
              end;

              if (Kill_ConsecLoss > 0) and (lossesSeguidas >= Kill_ConsecLoss) then
              begin
                travaDia        := True;
                auditProtReason := "KILL_CONSECLOSS";
              end;

              auditEfficiency  := auditPnL / auditMFE;
              if auditEfficiency < 0 then auditEfficiency := 0;
              if auditEfficiency > 1 then auditEfficiency := 1;

              auditExitReason  := xaiExitCause;
              auditEntryReason := xaiSignalReason;

              regimeAdjExit := 0.55 + (volatilityState * 0.06);
              if (regimeMercado = 2) then regimeAdjExit := regimeAdjExit + 0.10;
              if (regimeMercado = 1) then regimeAdjExit := regimeAdjExit - 0.04;
              if (regimeAdjExit < 0.45) then regimeAdjExit := 0.45;
              if (regimeAdjExit > 0.80) then regimeAdjExit := 0.80;

              entropySignal := 0.30 + (volatilityState * 0.08);
              if (regimeMercado = 2) then entropySignal := entropySignal + 0.04;
              if (entropySignal < 0.22) then entropySignal := 0.22;
              if (entropySignal > 0.55) then entropySignal := 0.55;

              auditEffNorm  := auditEfficiency / Max(0.0001, regimeAdjExit);
              if auditEffNorm < 0 then auditEffNorm := 0;
              if auditEffNorm > 1 then auditEffNorm := 1;

              auditPnLNorm  := auditPnL_ATR / Max(0.0001, entropySignal);
              if auditPnLNorm < 0 then auditPnLNorm := 0;
              if auditPnLNorm > 1 then auditPnLNorm := 1;

              auditPainNorm := 1 - Min(1, auditMAE / Max(0.0001, auditMFE));
              if auditPainNorm < 0 then auditPainNorm := 0;
              if auditPainNorm > 1 then auditPainNorm := 1;

              confHi := 0.55;
              confLo := 0.35;

              if (regimeMercado = 3) then
              begin
                confHi := 0.45;
                confLo := 0.30;
              end
              else if (regimeMercado = 0) then
              begin
                confHi := 0.60;
                confLo := 0.30;
              end
              else if (volatilityState = 2) then
              begin
                confHi := 0.50;
                confLo := 0.25;
              end;

              auditScore := Round(100 * (
                             (confHi * auditEffNorm) +
                             (confLo * auditPnLNorm) +
                             ((1.0 - confHi - confLo) * auditPainNorm)
                           ));
              if auditScore < 0 then auditScore := 0;
              if auditScore > 100 then auditScore := 100;

              if (not yellowCaptureOk) then
                auditScore := Max(0, auditScore - Round(10 + (yellowGivebackATR * 10)));

              if (auditPnL_ATR < 0.0) then auditTag := "RUIM"
              else if (not yellowCaptureOk) then auditTag := "C_CAPTURA"
              else if (auditScore >= 85) then auditTag := "A"
              else if (auditScore >= 70) then auditTag := "B"
              else auditTag := "C";

              // ==============================
              // WALK-FORWARD / FORA DA AMOSTRA
              // ==============================
              if WF_Ativo then
              begin
                // Classifica o trade fechado pela BARRA DE SAÍDA, não pela barra de entrada.
                // Isso evita vazamento de informação quando um trade nasce no treino e fecha na OOS.
                if (((auditExitBar - (Floor(auditExitBar / wfCycleBars) * wfCycleBars)) < WF_BarrasTreino) and
                    (Floor(auditExitBar / wfCycleBars) = wfFoldId)) then
                begin
                  wfTrainTrades := wfTrainTrades + 1;
                  wfTrainEquity_ATR := wfTrainEquity_ATR + auditPnL_ATR;
                  if (wfTrainEquity_ATR > wfTrainPeak_ATR) then
                    wfTrainPeak_ATR := wfTrainEquity_ATR;

                  wfTrainDD_ATR := wfTrainPeak_ATR - wfTrainEquity_ATR;
                  if (wfTrainDD_ATR > wfTrainMaxDD_ATR) then
                    wfTrainMaxDD_ATR := wfTrainDD_ATR;

                  if (auditPnL_ATR > 0) then
                  begin
                    wfTrainWins := wfTrainWins + 1.0;
                    wfTrainGain_ATR := wfTrainGain_ATR + auditPnL_ATR;
                  end
                  else
                    wfTrainLoss_ATR := wfTrainLoss_ATR + Abs(auditPnL_ATR);

                  wfTrainWinRate := wfTrainWins / Max(1, wfTrainTrades);
                  wfTrainExp_ATR := wfTrainEquity_ATR / Max(1, wfTrainTrades);
                  wfTrainPF      := wfTrainGain_ATR / Max(0.0001, wfTrainLoss_ATR);
                end
                else
                begin
                  if (Floor(entryBarIdx / wfCycleBars) <> Floor(auditExitBar / wfCycleBars)) and (wfBlockReason = "") then
                    wfBlockReason := "WF=CARRY_TRADE";

                  wfOOSTrades := wfOOSTrades + 1;
                  wfOOSEquity_ATR := wfOOSEquity_ATR + auditPnL_ATR;
                  if (wfOOSEquity_ATR > wfOOSPeak_ATR) then
                    wfOOSPeak_ATR := wfOOSEquity_ATR;

                  wfOOSDD_ATR := wfOOSPeak_ATR - wfOOSEquity_ATR;
                  if (wfOOSDD_ATR > wfOOSMaxDD_ATR) then
                    wfOOSMaxDD_ATR := wfOOSDD_ATR;

                  if (auditPnL_ATR > 0) then
                  begin
                    wfOOSWins := wfOOSWins + 1.0;
                    wfOOSGain_ATR := wfOOSGain_ATR + auditPnL_ATR;
                  end
                  else
                    wfOOSLoss_ATR := wfOOSLoss_ATR + Abs(auditPnL_ATR);

                  wfOOSWinRate := wfOOSWins / Max(1, wfOOSTrades);
                  wfOOSExp_ATR := wfOOSEquity_ATR / Max(1, wfOOSTrades);
                  wfOOSPF      := wfOOSGain_ATR / Max(0.0001, wfOOSLoss_ATR);
                end;
              end;

              // ==============================
              // MÉTRICAS ADAPTATIVAS (EWMA) + PROTEÇÃO
              // ==============================
              if (auditTrades = 0) then
              begin
                if (auditPnL_ATR > 0) then
                  auditEwmaWin := 1.0
                else
                  auditEwmaWin := 0.0;

                if (auditPnL_ATR > 0) then
                  auditEwmaGain_ATR := auditPnL_ATR
                else
                  auditEwmaGain_ATR := 0.0;

                if (auditPnL_ATR < 0) then
                  auditEwmaLoss_ATR := Abs(auditPnL_ATR)
                else
                  auditEwmaLoss_ATR := 0.0;

                auditEwmaEff := auditEfficiency;

                // Correção-Σ:
                // Não carregar auditPnL_ATR aqui.
                // A soma oficial acontece uma única vez no bloco:
                // auditEquity_ATR := auditEquity_ATR + auditPnL_ATR;
                auditEquity_ATR := 0.0;
                auditPeak_ATR   := 0.0;
                auditDD_ATR     := 0.0;
                auditMaxDD_ATR  := 0.0;

                auditProtBarsLeft := 0;
                auditProtReason   := "";
              end;

              auditTrades := auditTrades + 1;

              auditAlpha := 0.08 + (volatilityState * 0.04) + (Abs(marketMomentum) * 0.03);
              if auditAlpha < 0.05 then auditAlpha := 0.05;
              if auditAlpha > 0.25 then auditAlpha := 0.25;

              if (auditPnL_ATR > 0) then
                auditEwmaWin := ((1 - auditAlpha) * auditEwmaWin) + (auditAlpha * 1)
              else
                auditEwmaWin := ((1 - auditAlpha) * auditEwmaWin);

              if (auditPnL_ATR > 0) then
                auditEwmaGain_ATR := ((1 - auditAlpha) * auditEwmaGain_ATR) + (auditAlpha * auditPnL_ATR)
              else
                auditEwmaGain_ATR := ((1 - auditAlpha) * auditEwmaGain_ATR);

              if (auditPnL_ATR < 0) then
                auditEwmaLoss_ATR := ((1 - auditAlpha) * auditEwmaLoss_ATR) + (auditAlpha * Abs(auditPnL_ATR))
              else
                auditEwmaLoss_ATR := ((1 - auditAlpha) * auditEwmaLoss_ATR);

              auditEwmaEff := ((1 - auditAlpha) * auditEwmaEff) + (auditAlpha * auditEfficiency);

              auditExpectancy_ATR := auditEwmaGain_ATR - auditEwmaLoss_ATR;
              auditProfitFactor   := auditEwmaGain_ATR / Max(0.0001, auditEwmaLoss_ATR);

              auditEquity_ATR := auditEquity_ATR + auditPnL_ATR;
              if (auditEquity_ATR > auditPeak_ATR) then auditPeak_ATR := auditEquity_ATR;
              auditDD_ATR := auditPeak_ATR - auditEquity_ATR;
              if (auditDD_ATR > auditMaxDD_ATR) then auditMaxDD_ATR := auditDD_ATR;

              if (auditTrades >= 8) then
              begin
                regimeAdjExit := 0.42 - (volatilityState * 0.04);
                if (regimeAdjExit < 0.28) then regimeAdjExit := 0.28;
                if (regimeAdjExit > 0.46) then regimeAdjExit := 0.46;

                entropySignal := 2.20 + (volatilityState * 0.60);
                if (entropySignal < 1.80) then entropySignal := 1.80;
                if (entropySignal > 3.80) then entropySignal := 3.80;

                if (auditDD_ATR > entropySignal) or ((auditExpectancy_ATR < 0) and (auditEwmaWin < regimeAdjExit) and (auditProfitFactor < 1.0)) then
                begin
                  auditProtBarsLeft := Round(3 + (2 * volatilityState));
                  if auditProtBarsLeft < 3 then auditProtBarsLeft := 3;
                  if auditProtBarsLeft > 9 then auditProtBarsLeft := 9;

                  if (auditDD_ATR > entropySignal) then
                    auditProtReason := "PROTECT:DD_HIGH"
                  else
                    auditProtReason := "PROTECT:EXPECT_NEG";

                  if (auditTag = "RUIM") and (auditProtBarsLeft < (4 + volatilityState)) then
                    auditProtBarsLeft := (4 + volatilityState);
                end
                else
                begin
                  if (auditProtBarsLeft = 0) then auditProtReason := "";
                end;
              end;

            // ==============================
            // RESET DO CICLO
            // ==============================
             lastEntrySideExit := lastEntrySide;

             if SomenteSinais then
              begin
                holdSell      := False;
                lastEntrySide := 0;

                // RESET PÉTREO PÓS-AMARELO:
                // Este bloco só executa dentro de optimalExit, portanto o candle atual já é AMARELO.
                // O próximo candle deve ser NEUTRO obrigatório.
                resetBars := 1;
              end

              else
              begin
                exitPending     := True;
                exitPendingSide := -1;
                exitSignalBar   := CurrentBar();
                resetBars       := 0;
              end;
            end
            
            else
            begin
              // PERSISTÊNCIA DO VERMELHO
              paintStateSeries[0] := -1;
            end;
          end;
        end;
      end;
  
     // ==================== ENGINE DE EXECUÇÃO (CONTRATO FORMAL AMARELO ↔ FILL REAL) — Σ ====================
    // Regras:
    // 1) Em execução real, a posição da automação é soberana.
    // 2) O AMARELO visual só fecha o ciclo depois do flat confirmado.
    // 3) Enquanto houver posição + saída pendente, o estado fica travado em AMARELO.
    // 4) Nunca reentra antes do FLAT confirmado + 1 candle neutro.
    // 5) Nunca duplica entrada enquanto existir ordem pendente.
    // 6) Em gap/evento extremo, aumenta offset e reduz ilusão de proteção.
    if (not SomenteSinais) then
    begin
      // ===== CONTRATO FORMAL AMARELO ↔ FILL REAL =====
      if exitPending then
      begin
        paintStateSeries[0] := 2;
        gateOpen         := False;
        allowTrade       := False;
        allowTradeGlobal := False;
        adaptiveGateOk   := False;
        entryPending     := False;
        entryPendingSide := 0;
        entrySignalBar   := -1;

        exitPendingBars := CurrentBar() - exitSignalBar;
        if (exitPendingBars < 0) then exitPendingBars := 0;

        if (xaiExitCause = "") then
          xaiExitCause := "EXIT=PENDENTE_FILL_REAL";

        if HasPosition then
        begin
          qtyExitEff := Max(1.0, Abs(PositionQty));

          // Se o fill não confirma rápido, reenvia com cancelamento prévio.
          if (CurrentBar() > exitSignalBar) or (exitPendingBars >= Exec_MaxPendingBars) then
          begin
            if HasPendingOrders then
              CancelPendingOrders;
            if IsBought then
              SellToCoverAtMarket(qtyExitEff);
            if IsSold then
              BuyToCoverAtMarket(qtyExitEff);
            // Limpa estados de posição após execução
            holdBuy  := False;
            holdSell := False;

            exitSignalBar := CurrentBar();
          end;

          if (exitPendingBars > Exec_MaxPendingBars) then
          begin
            auditProtActive   := True;
            auditProtBarsLeft := Max(auditProtBarsLeft, 2);
            auditProtReason   := "EXEC:FILL_SAIDA_ATRASADO";
            xaiBlockReason    := "EXEC:FILL_SAIDA_ATRASADO";
          end;
        end
        else
        begin
          // FLAT confirmado: somente aqui o ciclo pode armar reset neutro.
          exitPending     := False;
          exitPendingSide := 0;
          exitSignalBar   := -1;
          resetBars       := 1;
          slPrice         := 0;
          highSinceEntry  := 0;
          lowSinceEntry   := 0;
        end;
      end

      // FLAT -> só entra se não houve AMARELO na barra anterior,
      // se não há reset em andamento, se não existe saída pendente,
      // se não existe ordem pendente e se a produção foi promovida.
      else if (not HasPosition) then
      begin
        if HasPendingOrders then
        begin
          entryPending := True;

          if (entryPendingSide = 0) then
            entryPendingSide := paintStateSeries[0];

          if (entrySignalBar < 0) then
            entrySignalBar := CurrentBar();

          entryPendingBars := CurrentBar() - entrySignalBar;
          if (entryPendingBars < 0) then entryPendingBars := 0;

          if (entryPendingBars > Exec_MaxPendingBars) then
          begin
            CancelPendingOrders;
            entryPending     := False;
            entryPendingSide := 0;
            entrySignalBar   := -1;
            paintStateSeries[0] := 0;
            xaiBlockReason := "EXEC:ENTRADA_CANCELADA_SEM_FILL";
          end;
        end
        else
        begin
          entryPending     := False;
          entryPendingSide := 0;
          entrySignalBar   := -1;

          if ((CurrentBar() = 0) or (paintStateSeries[1] <> 2)) and
             (resetBars = 0) and
             allowTradeGlobal and adaptiveGateOk and
             wfGateOk and
             ((not Prod_BloquearExecucaoReal) or prodReady or SomenteSinais) then

          begin
            if (paintStateSeries[0] = 1) then
            begin
              BuyAtMarket(qtyEntryEff);
              entryPending     := True;
              entryPendingSide := 1;
              entrySignalBar   := CurrentBar();
            end
            else if (paintStateSeries[0] = -1) then
            begin
              SellShortAtMarket(qtyEntryEff);
              entryPending     := True;
              entryPendingSide := -1;
              entrySignalBar   := CurrentBar();
            end;
          end
          else
          begin
            if (paintStateSeries[0] = 1) or (paintStateSeries[0] = -1) then
            begin
              paintStateSeries[0] := 0;

              if prodBlock and (prodReason <> "") then
                xaiBlockReason := prodReason
              else if (not wfGateOk) then
                xaiBlockReason := "EXEC:WF_GATE_OFF"
              else if (not allowTradeGlobal) then
                xaiBlockReason := "EXEC:ALLOW_GLOBAL_OFF"
              else if (not adaptiveGateOk) then
                xaiBlockReason := "EXEC:ADAPTIVE_GATE_OFF"
              else
                xaiBlockReason := "EXEC:ENTRADA_BLOQUEADA_SEM_FILL";
            end;
          end;
        end;
      end

      // COM POSIÇÃO -> ou protege, ou inicia a saída do AMARELO.
      else
      begin
        if entryPending then
        begin
          tradesHoje := tradesHoje + 1;
          if (MaxTradesDia > 0) and (tradesHoje >= MaxTradesDia) then
          begin
            travaDia        := True;
            auditProtReason := "MAXTRADES_DIA";
          end;
        end;

        entryPending     := False;
        entryPendingSide := 0;
        entrySignalBar   := -1;

        qtyExitEff := Max(1.0, Abs(PositionQty));

        if (paintStateSeries[0] = 2) then
        begin
          exitPending     := True;
          exitPendingSide := lastEntrySide;
          exitSignalBar   := CurrentBar();

          yellowVisualPx := Close[0];

          // Cláusula pétrea do AMARELO executável:
          // antes de zerar a mercado, elimina OCO/stop/limit pendente para evitar ordem fantasma.
          if HasPendingOrders then
            CancelPendingOrders;

          if IsBought then
            SellToCoverAtMarket(qtyExitEff);

          if IsSold then
            BuyToCoverAtMarket(qtyExitEff);
        end
        else if Usar_OCO_ToCover and (slPrice > 0) then
        begin
          // Atualização limpa de proteção: evita empilhar stops antigos.
          if HasPendingOrders then
            CancelPendingOrders;

          stopOffsetTicksEff := Exec_StopOffsetMin_Ticks;

          if Exec_UsarCustosReais then
            stopOffsetTicksEff := Max(stopOffsetTicksEff,
                                      Exec_Spread_Ticks + Exec_Delay_Ticks + Exec_SlippageOut_Ticks);

          if (volatilityState = 2) then
            stopOffsetTicksEff := stopOffsetTicksEff + Max(2.0, Exec_PenalidadeVolAlta_Ticks);

          if extremeGapActive then
            stopOffsetTicksEff := stopOffsetTicksEff + Exec_GapEmergency_Ticks;

          stopOffsetTicksEff := stopOffsetTicksEff + ((Exec_StopOffsetATR_Fator * atr0) / Max(0.0001, MinPriceIncrement));
          targetOffset := stopOffsetTicksEff * MinPriceIncrement;

          if IsBought then
            SellToCoverStop(slPrice, slPrice - targetOffset, qtyExitEff);

          if IsSold then
            BuyToCoverStop(slPrice, slPrice + targetOffset, qtyExitEff);
        end;
      end;
    end;  

    // ==============================
    // SUÍTE DE REGRESSÃO PÉTREA AUTOMÁTICA
    // 1) uma e somente uma cor por candle
    // 2) nunca hold duplo
    // 3) amarelo exige causa real, lado real e armar reset
    // 4) pós-amarelo exige 1 candle neutro
    // 5) stop em vol alta exige slPrice real
    // 6) topo+reversão / fundo+repique exigem amarelo na própria barra válida
    // 7) consolida cenários vistos + falhas acumuladas em uma suíte única
    // ==============================

    // Integridade do estado final (uma cor por candle)
    if (paintStateSeries[0] <> -1) and
       (paintStateSeries[0] <> 0) and
       (paintStateSeries[0] <> 1) and
       (paintStateSeries[0] <> 2) then
    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 4);
      auditProtReason   := "VALIDACAO:ESTADO_COR_INVALIDO";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:ESTADO_COR_INVALIDO";

      if ModoDebug then
        PlotText("FALHA:ESTADO_COR_INVALIDO", clRed, 1, 8, High[0] + 0.30 * atr0);
    end;

    // Estado impossível: comprado e vendido ao mesmo tempo
    if holdBuy and holdSell then
    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 6);
      auditProtReason   := "VALIDACAO:HOLD_DUPLO";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:HOLD_DUPLO";

      if ModoDebug then
        PlotText("FALHA:HOLD_DUPLO", clRed, 1, 8, High[0] + 0.38 * atr0);
    end;

    // AMARELO precisa ter causa real explícita e EXECUTÁVEL.
    // Motivo BLOCK não é causa de saída; é motivo de bloqueio.
    if (paintStateSeries[0] = 2) and
       (
         (xaiExitCause = "") or
         (xaiExitCause = "BLOCK:VOL_NOISE_WEAKREV_BUY") or
         (xaiExitCause = "BLOCK:VOL_SPIKE_CONT_BUY") or
         (xaiExitCause = "BLOCK:VOL_LEVELTEST_WEAKREV_BUY") or
         (xaiExitCause = "BLOCK:BUY_REVERSAL_NOT_EXECUTABLE") or
         (xaiExitCause = "BLOCK:VOL_NOISE_WEAKREV_SELL") or
         (xaiExitCause = "BLOCK:VOL_SPIKE_CONT_SELL") or
         (xaiExitCause = "BLOCK:VOL_LEVELTEST_WEAKREV_SELL") or
         (xaiExitCause = "BLOCK:SELL_REVERSAL_NOT_EXECUTABLE")
       ) then
    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 6);
      auditProtReason   := "VALIDACAO:AMARELO_SEM_CAUSA_EXECUTAVEL";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:AMARELO_SEM_CAUSA_EXECUTAVEL";

      if ModoDebug then
        PlotText("FALHA:AMARELO_CAUSA_NAO_EXECUTAVEL", clRed, 1, 8, High[0] + 0.44 * atr0);
    end;

    // AMARELO precisa carregar o lado que acabou de ser fechado
    if (paintStateSeries[0] = 2) and (lastEntrySideExit = 0) then
    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 6);
      auditProtReason   := "VALIDACAO:AMARELO_SEM_LADO";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:AMARELO_SEM_LADO";

      if ModoDebug then
        PlotText("FALHA:AMARELO_SEM_LADO", clRed, 1, 8, High[0] + 0.50 * atr0);
    end;

    // AMARELO EXECUTÁVEL:
    // no protocolo canônico deste sistema, o AMARELO nasce justamente
    // na barra de encerramento do trade. Portanto, coexistência transitória
    // com o lado recém-fechado NÃO é falha estrutural.
    //
    // As falhas reais já são cobertas pelos validadores:
    // - HOLD_DUPLO
    // - AMARELO_SEM_LADO
    // - AMARELO_SEM_ARMAR_RESET
    //
    // Este bloco permanece intencionalmente neutro para não invalidar
    // um AMARELO legítimo e não atrasar realização de lucro.
    
    // AMARELO precisa respeitar o contrato de reset:
    // - SomenteSinais: arma reset imediatamente.
    // - Execução real: primeiro fecha de verdade, depois arma 1 candle neutro.
    if (paintStateSeries[0] = 2) then
    begin
      if (SomenteSinais and (resetBars <> 1)) or
         ((not SomenteSinais) and (not exitPending) and (resetBars <> 1)) then
      begin
        auditProtActive   := True;
        auditProtBarsLeft := Max(auditProtBarsLeft, 5);
        auditProtReason   := "VALIDACAO:AMARELO_SEM_ARMAR_RESET";
        gateOpen          := False;
        allowTrade        := False;
        adaptiveGateOk    := False;
        xaiBlockReason    := "FALHA_VALIDACAO:AMARELO_SEM_ARMAR_RESET";

        if ModoDebug then
          PlotText("FALHA:AMARELO_SEM_ARMAR_RESET", clRed, 1, 8, High[0] + 0.62 * atr0);
      end;
    end;

    // Lado incoerente do amarelo
    // Se o trade fechado foi BUY, nenhuma causa da família SELL pode sobreviver.
    if (paintStateSeries[0] = 2) and (lastEntrySideExit = 1) and
       (
         (xaiExitCause = "EXIT=PO_MIN_EXECUTAVEL") or
         (xaiExitCause = "EXIT=SELL_REVERSAL_LOW_RISK") or
         (xaiExitCause = "EXIT=SELL_FIRST_REV_AFTER_BOTTOM_LOCK") or
         (xaiExitCause = "EXIT=SELL_FIRST_EXEC_REV_NO_OPPOSITE_WAIT") or
         (xaiExitCause = "EXIT=STRUCT_REV_AFTER_BOTTOM_EXEC") or
         (xaiExitCause = "EXIT=POST_BOTTOM_CONFIRM_EXEC") or
         (xaiExitCause = "EXIT=EXTREME_MICROREV_SELL") or
         (xaiExitCause = "EXIT=SELL_REVERSAL_EDGE_EXEC") or
         (xaiExitCause = "EXIT=CONFIRM_XAI_AFTER_EXTREME_SELL") or
         (xaiExitCause = "EXIT=TIME_DECAY_PROFIT_LOCK_SELL") or
         (xaiExitCause = "EXIT=EXHAUST_WICK(+BB)+PROFIT_OK_SELL") or
         (xaiExitCause = "EXIT=STOP(SLPRICE_REAL_SELL)") or
         (xaiExitCause = "EXIT=STOP_REAL_SELL")
       ) then

    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 6);
      auditProtReason   := "VALIDACAO:LADO_AMARELO_INCOERENTE_BUY";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:LADO_AMARELO_INCOERENTE_BUY";

      if ModoDebug then
        PlotText("FALHA:LADO_AMARELO_BUY", clRed, 1, 8, High[0] + 0.68 * atr0);
    end;

    // Se o trade fechado foi SELL, nenhuma causa da família BUY pode sobreviver.
    if (paintStateSeries[0] = 2) and (lastEntrySideExit = -1) and
       (
         (xaiExitCause = "EXIT=PO_MAX_EXECUTAVEL") or
         (xaiExitCause = "EXIT=BUY_REVERSAL_LOW_RISK") or         
         (xaiExitCause = "EXIT=BUY_FIRST_REV_AFTER_TOP_LOCK") or
         (xaiExitCause = "EXIT=BUY_FIRST_EXEC_REV_NO_OPPOSITE_WAIT") or         
         (xaiExitCause = "EXIT=STRUCT_REV_AFTER_TOP_EXEC") or
         (xaiExitCause = "EXIT=POST_TOP_CONFIRM_EXEC") or
         (xaiExitCause = "EXIT=EXTREME_MICROREV_BUY") or
         (xaiExitCause = "EXIT=BUY_REVERSAL_EDGE_EXEC") or
         (xaiExitCause = "EXIT=CONFIRM_XAI_AFTER_EXTREME_BUY") or
         (xaiExitCause = "EXIT=TIME_DECAY_PROFIT_LOCK_BUY") or
         (xaiExitCause = "EXIT=EXHAUST_WICK(+BB)+PROFIT_OK") or
         (xaiExitCause = "EXIT=STOP(SLPRICE_REAL_BUY)") or
         (xaiExitCause = "EXIT=STOP_REAL_BUY")
       ) then
    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 6);
      auditProtReason   := "VALIDACAO:LADO_AMARELO_INCOERENTE_SELL";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:LADO_AMARELO_INCOERENTE_SELL";

      if ModoDebug then
        PlotText("FALHA:LADO_AMARELO_SELL", clRed, 1, 8, High[0] + 0.74 * atr0);
    end;

    // Pós-amarelo: a barra seguinte precisa nascer neutra
    // EXCETO quando a saída real ainda está pendente e a posição não zerou.
    if (statePrev = 2) and (paintStateSeries[0] <> 0) and
       (
         SomenteSinais or
         ((not SomenteSinais) and (not exitPending) and (not HasPosition))
       ) then
    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 5);
      auditProtReason   := "VALIDACAO:SEM_RESET_NEUTRO";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:SEM_RESET_NEUTRO";
      regFailCount      := regFailCount + 1;

      if ModoDebug then
        PlotText("FALHA:SEM_RESET_NEUTRO", clRed, 1, 8, High[0] + 0.80 * atr0);
    end;

    // Stop real em vol alta: aceita as duas nomenclaturas válidas
    if exitByStop and (volatilityState = 2) and
       (
         (slPrice <= 0) or
         (
           (xaiExitCause <> "EXIT=STOP(SLPRICE_REAL_BUY)") and
           (xaiExitCause <> "EXIT=STOP(SLPRICE_REAL_SELL)") and
           (xaiExitCause <> "EXIT=STOP_REAL_BUY") and
           (xaiExitCause <> "EXIT=STOP_REAL_SELL")
         )
       ) then

    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 6);
      auditProtReason   := "VALIDACAO:STOP_NAO_SLP";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:STOP_NAO_SLP";
      regFailCount      := regFailCount + 1;

      if ModoDebug then
        PlotText("FALHA:STOP_NAO_SLP", clRed, 1, 8, High[0] + 0.86 * atr0);
    end
    else if exitByStop and (volatilityState = 2) then
      regSeenStopVol := True;

    // Compra: topo + reversão material precisa virar amarelo na própria barra válida
    if holdBuy and
       (
         // NOVO: valida também o topo no próprio candle extremo
         (
           (offsetMaxima = 0) and
           (maxGainATR >= Max(0.12, minGainExitATR * 0.45)) and
           (upperWickCurrent >= Max(0.28, Min(0.52, ratioThresh + 0.02))) and
           (
             condition4 or
             (Close[0] < Open[0]) or
             (Close[0] <= ((currentHigh + currentLow) * 0.50))
           )
         )
         or
         // REGRA NOVA: somente a PRIMEIRA barra pós-topo é válida
         (
           (offsetMaxima = 1) and
           (maxGainATR >= Max(0.12, minGainExitATR * 0.50)) and
           (
             (Close[0] < Close[1]) or
             (currentLow <= Low[offsetMaxima]) or
             (
               (High[0] <= High[offsetMaxima]) and
               (
                 condition4 or
                 (Close[0] <= ((High[offsetMaxima] + Low[offsetMaxima]) * 0.55))
               )
             )
           )
         )
       ) and
       (paintStateSeries[0] <> 2) then
    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 4);
      auditProtReason   := "VALIDACAO:BUY_TOPREV_NO_YELLOW";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:BUY_TOPREV_NO_YELLOW";
      regFailCount      := regFailCount + 1;

      if ModoDebug then
        PlotText("FALHA:BUY_TOPREV_NO_YELLOW", clRed, 1, 8, High[0] + 0.92 * atr0);
    end

    else if (lastEntrySideExit = 1) and
            (
              (xaiExitCause = "EXIT=PO_MAX_EXECUTAVEL") or
              (xaiExitCause = "EXIT=BUY_REVERSAL_LOW_RISK") or
              (xaiExitCause = "EXIT=BUY_FIRST_REV_AFTER_TOP_LOCK") or
              (xaiExitCause = "EXIT=BUY_FIRST_EXEC_REV_NO_OPPOSITE_WAIT") or
              (xaiExitCause = "EXIT=STRUCT_REV_AFTER_TOP_EXEC") or
              (xaiExitCause = "EXIT=POST_TOP_CONFIRM_EXEC") or
              (xaiExitCause = "EXIT=EXTREME_MICROREV_BUY") or
              (xaiExitCause = "EXIT=BUY_REVERSAL_EDGE_EXEC") or
              (xaiExitCause = "EXIT=CONFIRM_XAI_AFTER_EXTREME_BUY") or
              (xaiExitCause = "EXIT=CONFIRM(XAI_SCORE)") or
              (xaiExitCause = "EXIT=TIME_DECAY_PROFIT_LOCK_BUY") or
              (xaiExitCause = "EXIT=EXHAUST_WICK(+BB)+PROFIT_OK")
            ) and (paintStateSeries[0] = 2) then
      regSeenBuyTopRev := True;

    // Venda: fundo + repique material precisa virar amarelo na própria barra válida
    if holdSell and
       (
         // NOVO: valida também o fundo no próprio candle extremo
         (
           (offsetMinima = 0) and
           (maxGainATR >= Max(0.12, minGainExitATR * 0.45)) and
           (lowerWickCurrent >= Max(0.28, Min(0.52, ratioThresh + 0.02))) and
           (
             condition4 or
             (Close[0] > Open[0]) or
             (Close[0] >= ((currentHigh + currentLow) * 0.50))
           )
         )
         or
         // REGRA NOVA: somente a PRIMEIRA barra pós-fundo é válida
         (
           (offsetMinima = 1) and
           (maxGainATR >= Max(0.12, minGainExitATR * 0.50)) and
           (
             (Close[0] > Close[1]) or
             (currentHigh >= High[offsetMinima]) or
             (
               (Low[0] >= Low[offsetMinima]) and
               (
                 condition4 or
                 (Close[0] >= ((High[offsetMinima] + Low[offsetMinima]) * 0.45))
               )
             )
           )
         )
       ) and
       (paintStateSeries[0] <> 2) then
    begin
      auditProtActive   := True;
      auditProtBarsLeft := Max(auditProtBarsLeft, 4);
      auditProtReason   := "VALIDACAO:SELL_BOTTOMREV_NO_YELLOW";
      gateOpen          := False;
      allowTrade        := False;
      adaptiveGateOk    := False;
      xaiBlockReason    := "FALHA_VALIDACAO:SELL_BOTTOMREV_NO_YELLOW";
      regFailCount      := regFailCount + 1;

      if ModoDebug then
        PlotText("FALHA:SELL_BOTTOMREV_NO_YELLOW", clRed, 1, 8, High[0] + 0.98 * atr0);
    end

    else if (lastEntrySideExit = -1) and
            (
              (xaiExitCause = "EXIT=PO_MIN_EXECUTAVEL") or
              (xaiExitCause = "EXIT=SELL_REVERSAL_LOW_RISK") or
              (xaiExitCause = "EXIT=SELL_FIRST_REV_AFTER_BOTTOM_LOCK") or
              (xaiExitCause = "EXIT=SELL_FIRST_EXEC_REV_NO_OPPOSITE_WAIT") or
              (xaiExitCause = "EXIT=STRUCT_REV_AFTER_BOTTOM_EXEC") or
              (xaiExitCause = "EXIT=POST_BOTTOM_CONFIRM_EXEC") or
              (xaiExitCause = "EXIT=EXTREME_MICROREV_SELL") or
              (xaiExitCause = "EXIT=SELL_REVERSAL_EDGE_EXEC") or
              (xaiExitCause = "EXIT=CONFIRM_XAI_AFTER_EXTREME_SELL") or
              (xaiExitCause = "EXIT=CONFIRM(XAI_SCORE)") or
              (xaiExitCause = "EXIT=TIME_DECAY_PROFIT_LOCK_SELL") or
              (xaiExitCause = "EXIT=EXHAUST_WICK(+BB)+PROFIT_OK_SELL")
            ) and (paintStateSeries[0] = 2) then
      regSeenSellBottomRev := True;
 
    // Consolidação final da suíte de regressão pétrea automática
    if ModoDebug then
    begin
      if regSeenBuyTopRev and regSeenSellBottomRev and (regFailCount = 0) then
      begin
        if regSeenStopVol then
          PlotText("REG_OK_PETREA+STOP", clGreen, 1, 8, Low[0] - 0.28 * atr0)
        else
          PlotText("REG_OK_PETREA", clGreen, 1, 8, Low[0] - 0.28 * atr0);
      end
      else if (regFailCount > 0) then
        PlotText("REG_FAIL_PETREA", clRed, 1, 8, Low[0] - 0.28 * atr0);
    end;

    // ==================== ENGINE DE VISUALIZAÇÃO CIENTÍFICA ====================
    // PINTURA LIGADA À MÁQUINA DE ESTADOS (paintStateSeries)
    // Estados:
    //   0 = NEUTRO   → CINZA (clSilver)
    //   1 = COMPRA   → CorOpCompra
    //  -1 = VENDA    → CorOpVenda
    //   2 = SAÍDA    → CorSaida

    // REGRA DE OURO (determinismo): UMA cor por candle, sempre pelo estado FINAL.
    if (paintStateSeries[0] = 2) then
    begin
      // ================= SAÍDA (AMARELO) =================
      PaintBar(CorSaida);

      if ModoDebug then
      begin

        // DSS: AÇÃO objetiva (o que fazer) — bloco com begin/end (NTSL-safe)
        if (lastEntrySideExit = 1) then
        begin
          PlotText("AÇÃO: VENDER (REALIZAR)", CorSaida, 1, 6, High[0] + 0.30 * atr0);
        end
        else
        begin
          if (lastEntrySideExit = -1) then
          begin
            PlotText("AÇÃO: COMPRAR (REALIZAR)", CorSaida, 1, 6, High[0] + 0.30 * atr0);
          end
          else
          begin
            PlotText("AÇÃO: SAIR (REALIZAR)", CorSaida, 1, 6, High[0] + 0.30 * atr0);
          end;
        end;

        // XAI / DSS — cada IF com begin/end para evitar "ELSE solto"
        infoText := "";

        if (xaiExitCause <> "") then
          infoText := "MOT=" + xaiExitCause;

        if (xaiGuardNote <> "") then
        begin
          if (infoText <> "") then infoText := infoText + " | ";
          infoText := infoText + "PG=" + xaiGuardNote;
        end;

        if (xaiContext <> "") then
        begin
          if (infoText <> "") then infoText := infoText + " | ";
          infoText := infoText + "CTX=" + xaiContext;
        end;

        if (auditTag <> "") then
        begin
          if (infoText <> "") then infoText := infoText + " | ";
          infoText := infoText + "TAG=" + auditTag;
        end;

        if (auditExitReason <> "") then
        begin
          if (infoText <> "") then infoText := infoText + " | ";
          infoText := infoText + "AUD=" + auditExitReason;
        end;

        if (auditProtActive) then
        begin
          if (infoText <> "") then infoText := infoText + " | ";
          infoText := infoText + "PROT=ON:" + auditProtReason;
        end
        else
        begin
          if (infoText <> "") then infoText := infoText + " | ";
          infoText := infoText + "PROT=OFF";
        end;

        if (infoText <> "") then
          PlotText(infoText, CorSaida, 1, 6, High[0] + 0.12 * atr0);
      end;
    end

    else if (paintStateSeries[0] = 1) then

    begin
      // ================= COMPRA (VERDE) =================
      PaintBar(CorOpCompra);

      if ModoDebug then
      begin
        // DSS
        PlotText("AÇÃO: COMPRAR", CorOpCompra, 1, 6, Low[0] - 0.30 * atr0);

        // XAI: Motivo do sinal (entrada)
        if (xaiSignalReason <> "") then
          PlotText("MOTIVO: " + xaiSignalReason, CorOpCompra, 1, 6, Low[0] - 0.20 * atr0);

        // Contexto
        if (xaiContext <> "") then
          PlotText("CONTEXTO: " + xaiContext, clSilver, 1, 6, Low[0] - 0.08 * atr0);
      end;
    end

    else if (paintStateSeries[0] = -1) then
    begin
      // ================= VENDA (VERMELHO) =================
      PaintBar(CorOpVenda);

      if ModoDebug then
      begin
        // DSS
        PlotText("AÇÃO: VENDER", CorOpVenda, 1, 6, High[0] + 0.30 * atr0);

        // XAI: Motivo do sinal (entrada)
        if (xaiSignalReason <> "") then
          PlotText("MOTIVO: " + xaiSignalReason, CorOpVenda, 1, 6, High[0] + 0.20 * atr0);
        
        // Contexto (sempre completo e determinístico)
        if (xaiContext <> "") then
          PlotText("CONTEXTO: " + xaiContext, clSilver, 1, 6, High[0] + 0.08 * atr0);
      end;
    end
    else
    begin
      // ================= NEUTRO (PRATA) =================
      PaintBar(clSilver);

      if ModoDebug then
      begin
        // DSS: ação do neutro (estrutura decisão: aguardar)
        PlotText("AÇÃO: AGUARDAR", clSilver, 1, 6, High[0] + 0.30 * atr0);

        // XAI: motivo do bloqueio (por que ficou de fora)
        if (xaiBlockReason <> "") then
          PlotText("MOTIVO: " + xaiBlockReason, clSilver, 1, 6, High[0] + 0.20 * atr0);

        // Contexto (sempre completo e determinístico)
        if (xaiContext <> "") then
          PlotText("CONTEXTO: " + xaiContext, clSilver, 1, 6, High[0] + 0.08 * atr0);
      end;
    end;

    // FECHAMENTO CANÔNICO DO ESTADO DA BARRA
    // Necessário para a auditoria do reset neutro pós-amarelo.
    statePrev := paintStateSeries[0];
   end;
end;
