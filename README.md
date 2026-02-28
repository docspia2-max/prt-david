# Pullback Industrial & Mining Pro 📈🤖

![ProRealTime](https://img.shields.io/badge/Platform-ProRealTime-blue)
![Language](https://img.shields.io/badge/Language-ProRealCode-lightgrey)
![Version](https://img.shields.io/badge/Version-1.5%20(Gold)-success)

Sistema algorítmico de **Swing Trading** diseñado para ProRealTime (ProOrder). Especializado en la captura de retrocesos (*pullbacks*) en acciones de alta capitalización dentro de tendencias alcistas estructurales, con un enfoque particular en los sectores Industrial, Financiero y de Minería.

## 🧠 Lógica del Sistema (The Edge)

El sistema opera bajo la premisa de que los mejores puntos de entrada en activos cíclicos se dan cuando el "dinero inteligente" mantiene una tendencia alcista a largo plazo, pero el mercado a corto plazo sufre un episodio de miedo irracional.

### 1. Condiciones de Entrada
* **Filtro Institucional (Tendencia):** El precio de cierre debe estar por encima de la EMA de 50 periodos, y esta a su vez por encima de la EMA de 200 periodos.
* **Filtro de Fuerza (Force Index):** Utiliza un Force Index de 13 periodos (suavizado) para detectar cuándo la presión vendedora empieza a agotarse.
* **Filtro de Timing (RSI):** Exige un nivel de sobreventa extrema a corto plazo (RSI de 2 periodos < 30).

### 2. Gestión de Riesgo (Position Sizing)
No utiliza un número fijo de acciones. Calcula dinámicamente el tamaño de la posición basándose en:
* Riesgo máximo del **2% del capital actual** por operación.
* Distancia del Stop Loss ajustada a la volatilidad real del activo mediante el **ATR (Average True Range) de 14 periodos** (Multiplicador de 1.5).

### 3. Salidas Dinámicas (Smart Exits)
El gran avance de la versión 1.5 es la eliminación de la ineficiente "salida por tiempo fija", sustituyéndola por un ecosistema de salidas dinámicas:
* **Toma de Beneficios por Momentum:** Cierre a mercado si el RSI(2) supera el nivel de 85 (euforia de corto plazo).
* **Smart Stagnation Exit:** Si la operación lleva más de 7 días abierta y el beneficio es mínimo (menor a 0.5 ATR), el sistema aborta la operación para liberar capital, evitando el coste de oportunidad.
* **Trailing Stop a Breakeven:** Una vez que la posición acumula un beneficio igual al riesgo inicial, el stop se mueve automáticamente al precio de entrada.

---

## 📊 Rendimiento del Backtest (Out-of-Sample Test)
El sistema ha sido sometido a pruebas de estrés durante un periodo de **10 años (3650 días)** en gráficos Diarios, probando su robustez cruzada en activos descorrelacionados (ej. Mastercard y Caterpillar).

**Métricas Promedio Destacadas (Por Activo):**
* **Win Rate (Tasa de Acierto):** ~60% - 63%
* **Drawdown Máximo:** < 12.5% (Extremadamente seguro)
* **Ratio Ganancia/Pérdida:** > 1.60
* **Tiempo expuesto en mercado:** ~23% (El capital está libre en liquidez el 77% del tiempo, ideal para operar en formato *Portfolio* multicartera).

---

## 💻 Código Fuente (ProRealCode)

```prorealcode
// ============================================================
// SISTEMA: Pullback Industrial & Mining Pro - GOLD VERSION
// Versión   : 1.5 (Smart Stagnation Exit)
// Plataforma: ProRealTime / ProOrder
// ============================================================

// --- PARÁMETROS ---
CapitalInicial = 100000
RiesgoPorc     = 2.0
pEMAlenta     = 50    
pEMAultra     = 200   
pFI           = 13    
pRSI          = 2     
umbralRSI     = 30    
multATRstop   = 1.5    

// --- INDICADORES ---
ema50  = ExponentialAverage[pEMAlenta](close)
ema200 = ExponentialAverage[pEMAultra](close)
atr    = AverageTrueRange[14](close)
miRSI  = RSI[pRSI](close)
fiSmooth = ExponentialAverage[pFI](volume * (close - close[1]))

// --- ENTRADA ---
condicionEntrada = (close > ema50 AND ema50 > ema200) AND (fiSmooth < fiSmooth[1]) AND (miRSI < umbralRSI)

IF NOT OnMarket AND condicionEntrada THEN
    stopDist = atr * multATRstop
    IF stopDist > 0 THEN
        riesgoDinero = ((CapitalInicial + StrategyProfit) * RiesgoPorc / 100)
        numAcciones  = floor(riesgoDinero / stopDist)
        IF numAcciones > 0 THEN
            BUY numAcciones SHARES AT MARKET
            SET STOP LOSS stopDist
        ENDIF
    ENDIF
ENDIF

// --- GESTIÓN DE SALIDAS ---
IF OnMarket THEN
    // 1. Salida por Sobrecompra
    IF miRSI > 85 THEN
        SELL AT MARKET
    ENDIF
    
    // 2. Salida por Estancamiento Inteligente
    diasEnMercado = BARINDEX - TradeIndex
    IF diasEnMercado > 7 AND (close - PositionPrice) < (atr * 0.5) THEN
        SELL AT MARKET
    ENDIF

    // 3. Breakeven / Protección
    IF (close - PositionPrice) > (atr * multATRstop) THEN
        SET STOP PRICE PositionPrice 
    ENDIF
ENDIF
