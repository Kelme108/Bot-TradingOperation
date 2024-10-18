# **MetaTrader 5 Trading Bot**

Este projeto é um robô de trading para o MetaTrader 5, focado em operar no ativo **WDOX24** utilizando dados de candles de 1 minuto (M1). O robô faz uso de um modelo de aprendizado de máquina pré-treinado, indicadores técnicos e lógica de negociação para tomar decisões de compra e venda.

## **Funcionalidades**

- **Integração com MetaTrader 5**: O robô se conecta diretamente à plataforma MetaTrader 5 para coletar dados em tempo real e enviar ordens de compra e venda.
- **Indicadores Técnicos**: Utiliza indicadores como RSI e Médias Móveis Exponenciais (EMAF, EMAM, EMAS) para análise de tendência.
- **Modelo de Machine Learning**: Um modelo de aprendizado de máquina previamente treinado é utilizado para prever a direção do mercado e gerar sinais de compra ou venda.
- **Automação de Ordens**: O robô envia automaticamente ordens de compra e venda, definindo stops e targets baseados nas previsões e no preço atual.
  
## **Requisitos**

- **Python 3.x**
- Bibliotecas Python:
  - MetaTrader5
  - pandas
  - pandas_ta
  - numpy
  - joblib
  - scikit-learn
- **MetaTrader 5**: O MetaTrader 5 precisa estar instalado e configurado com permissões para negociação.

## **Configuração**

### 1. **Instalação de Dependências**

Antes de rodar o script, instale as dependências utilizando o seguinte comando:

```bash
pip install MetaTrader5 pandas pandas-ta joblib scikit-learn
```
## **2. Modelo de Machine Learning**

O modelo pré-treinado utilizado pelo robô deve estar salvo em um arquivo chamado **`modelWDO2024_past01.pkl`**. O modelo pode ser carregado diretamente no script para realizar previsões.

## **3. Parâmetros de Negociação**

No código, o símbolo do ativo e o timeframe (1 minuto) estão definidos como:

```python
symbol = "WDOX24"
timeframe = mt5.TIMEFRAME_M1  # 1 minuto
num_bars = 500  # Número de candles para carregar
```

## **Como o Robô Funciona**

1. **Conexão ao MetaTrader 5**  
   O robô se conecta à plataforma MetaTrader 5 e começa a receber dados de candles em tempo real.
   
2. **Cálculo de Indicadores**  
   Utiliza os preços de fechamento para calcular os indicadores técnicos **RSI** e **EMA**, que são utilizados como features para o modelo de machine learning.

3. **Previsão com Machine Learning**  
   O robô carrega um modelo treinado e usa as features técnicas para prever o comportamento do mercado.

4. **Execução de Ordens**  
   - **Compra**: Se a previsão e o indicador apontam uma tendência de alta, o robô envia uma ordem de compra.
   - **Venda**: Se a previsão aponta uma tendência de baixa, o robô envia uma ordem de venda.

5. **Repetição**  
   O robô repete o ciclo a cada 60 segundos, recalculando os indicadores, atualizando as previsões e decidindo se deve abrir ou fechar uma posição.


## **Estrutura do Código**

- **buy_order**: Função que envia uma ordem de compra.
- **sell_order**: Função que envia uma ordem de venda.
- **Loop Principal**: O robô coleta os dados do MetaTrader 5, calcula indicadores, faz previsões e toma decisões de compra ou venda automaticamente.

## **Executando o Robô**

Basta rodar o script Python. Certifique-se de que o MetaTrader 5 está corretamente configurado e que o ativo desejado está disponível.

```bash
python trading_bot.py
```

## **Notas**
- O robô está configurado para operar em mini índice (WDO) e faz uso de estratégias baseadas em aprendizado de máquina e indicadores técnicos.
- Certifique-se de que as permissões de negociação automática estão habilitadas no MetaTrader 5.
- Tenha cuidado com o volume de ordens enviadas e os parâmetros de risco, como Stop Loss e Take Profit, configurados nas funções de ordem.

## **Configurações Padrão de Ordens**
No código, as funções de envio de ordens são configuradas da seguinte maneira:

## **Função de Compra**
``` bash
def buy_order():
    lot = 1.0
    price = mt5.symbol_info_tick(symbol).ask
    request = {
        "action": mt5.TRADE_ACTION_DEAL,
        "symbol": symbol,
        "volume": lot,
        "type": mt5.ORDER_TYPE_BUY,
        "price": price,
        "sl": price - 100,
        "tp": price + 100,
        "deviation": 10,
        "magic": 123456,
        "comment": "Buy order by ML model",
        "type_time": mt5.ORDER_TIME_GTC,
        "type_filling": mt5.ORDER_FILLING_IOC,
    }
    result = mt5.order_send(request)
    if result.retcode == mt5.TRADE_RETCODE_DONE:
        print("Compra executada com sucesso")
    else:
        print("Falha ao executar a ordem de compra")
```
## **Função de Venda**
``` bash
def sell_order():
    lot = 1.0
    price = mt5.symbol_info_tick(symbol).bid
    request = {
        "action": mt5.TRADE_ACTION_DEAL,
        "symbol": symbol,
        "volume": lot,
        "type": mt5.ORDER_TYPE_SELL,
        "price": price,
        "sl": price + 100,
        "tp": price - 100,
        "deviation": 10,
        "magic": 123456,
        "comment": "Sell order by ML model",
        "type_time": mt5.ORDER_TIME_GTC,
        "type_filling": mt5.ORDER_FILLING_IOC,
    }
    result = mt5.order_send(request)
    if result.retcode == mt5.TRADE_RETCODE_DONE:
        print("Venda executada com sucesso")
    else:
        print("Falha ao executar a ordem de venda")
```

## **Loop Principal**
O robô repete o processo de coleta de dados, cálculo de indicadores e previsão a cada 60 segundos:
``` bash
while True:
    # Carregar dados de candles
    rates = mt5.copy_rates_from_pos(symbol, timeframe, 0, num_bars)
    
    # Cálculo de indicadores
    df = pd.DataFrame(rates)
    df['EMA'] = df['close'].ewm(span=20).mean()
    df['RSI'] = ta.rsi(df['close'], length=14)
    
    # Carregar modelo de machine learning
    model = joblib.load("modelWDO2024_past01.pkl")
    X = df[['EMA', 'RSI']].dropna()  # Features
    y_pred = model.predict(X)  # Previsão

    # Decisão de compra ou venda
    if y_pred[-1] == 1:  # Previsão de alta
        buy_order()
    elif y_pred[-1] == -1:  # Previsão de baixa
        sell_order()
    
    time.sleep(60)  # Espera de 60 segundos para o próximo ciclo
```
