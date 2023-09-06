<!-- TOC -->
  * [Algoinfra](#algoinfra)
    * [`/upload`](#upload)
      * [sample strategy file](#sample-strategy-file)
      * [Strategy Code Explanation](#strategy-code-explanation)
      * [sample config.json file](#sample-configjson-file)
* [Backtest](#backtest)
  * [Configuration Items](#configuration-items)
    * [enabled](#enabled)
    * [without_legend](#withoutlegend-)
    * [instruments](#instruments)
      * [market](#market)
        * [Available markets](#available-markets)
      * [exchange](#exchange)
        * [Available exchanges](#available-exchanges)
      * [symbols](#symbols)
      * [additional_datas](#additionaldatas)
      * [start_date](#startdate)
      * [end_date](#enddate)
      * [timeframes](#timeframes)
      * [data_format](#dataformat)
        * [Available data_formats](#available-dataformats)
      * [starting_capital](#startingcapital)
      * [leverage](#leverage)
      * [commission](#commission)
      * [instant_profit_withdraw](#instantprofitwithdraw)
      * [open_position_on_close_price](#openpositiononcloseprice)
      * [close_position_on_close_price](#closepositiononcloseprice)
      * [analyzers](#analyzers)
        * [Available analyzers](#available-analyzers)
      * [observers](#observers)
  * [Backtest Essential Notes](#backtest-essential-notes)
* [Parameter Optimization](#parameter-optimization)
  * [Configuration Items](#configuration-items-1)
    * [enabled](#enabled-1)
    * [search_space](#searchspace)
    * [optimization_method](#optimizationmethod)
      * [Available Methods](#available-methods)
      * [Essential Tips](#essential-tips)
    * [objectives](#objectives)
      * [Available Functions](#available-functions)
      * [Essential Tips](#essential-tips-1)
    * [directions](#directions)
      * [Essential Tips](#essential-tips-2)
    * [callbacks](#callbacks)
    * [n_trials](#ntrials)
      * [Essential Tips](#essential-tips-3)
    * [stats](#stats)
      * [Available Stats](#available-stats)
    * [save_results_per_iteration](#saveresultsperiteration)
      * [Essential Tips](#essential-tips-4)
    * [plots](#plots)
      * [Available Plots](#available-plots)
      * [Essential Tips](#essential-tips-5)
    * [`/error`](#error)
      * [some common errors](#some-common-errors)
    * [`/result/<file_name>`](#resultfilename)
  * [Data Service](#data-service)
    * [`/update`](#update)
    * [You can give the task_id to `/task/{task_id}` and check the state of the task.](#you-can-give-the-taskid-to-tasktaskid-and-check-the-state-of-the-task)
      * [forex-oanda](#forex-oanda)
      * [tse-tse](#tse-tse)
        * [for tickers|index you should set args differently](#for-tickersindex-you-should-set-args-differently)
      * [crypto-kucoin](#crypto-kucoin)
    * [`/batch-update`](#batch-update)
    * [`/data`](#data)
* [Money Management](#money-management)
  * [How to use?](#how-to-use)
    * [Initialization](#initialization)
    * [In the `next` method](#in-the-next-method)
  * [Types of Money Management](#types-of-money-management-)
    * [donguy_simple](#donguysimple)
    * [dealenbert](#dealenbert)
  * [creating new money management methods](#creating-new-money-management-methods)
* [FAQ](#faq)
  * [Why is server slow?](#why-is-server-slow)
* [Report Problem](#report-problem)
<!-- TOC -->


***
## Algoinfra
`bt.infra.goldscanco.com/docs`

### `/upload`

#### sample strategy file
```python
# ------- MAKE SURE TO IMPORT StrategyInterface ------- #
from strategy_interface import StrategyInterface

# ---- MAKE SURE TO INHERIT FROM StrategyInterface ---- #
class SampleStrategy(StrategyInterface):
  # -- USE args FOR THE PARAMETERS OF YOUR STRATEGY -- #
    args = {
    }

# ------------------- Do not change ----------------- #
    def __init__(self, params=None) -> None:
        super().__init__()
        if params is not None:
            unknown_params = [k for k in params if k not in self.args]
            if unknown_params:
                raise ValueError(
                    f"Unknown parameter(s) defined in search space: {', '.join(unknown_params)}"
                )
            self.args.update(params)
# ----------------- Change from now on -------------- #

        self.exit_signal = False

    def next(self):
        # check if we are in position or not
        if not self.position:
            if not self.exit_signal:
                # buy in first candle available
                self.buy(size=0.001)
                self.exit_signal = True
                
        else:
            if self.exit_signal:
                # close in the next one
                self.close()
                self.exit_signal = False

```
#### Strategy Code Explanation
This code is an example of a backtesting strategy implemented using the Backtrader library in Python. 
The strategy is called SampleStrategy, and it inherits from the StrategyInterface class, which is a custom interface that enforces a standardized structure for backtesting strategies.

It's important to note that to use the StrategyInterface class, **you must import it at the top of your code and inherit from it**. 
This is necessary to enforce the standardized structure and to access the standard functions provided by the StrategyInterface class which can help simplify your code.
By using the StrategyInterface class, you can benefit from its standardized structure and functions, which make it easier to integrate your strategy into a larger backtesting framework.
***
#### sample config.json file
```
{
  "strategy": {
    "name": "SampleStrategy",   ***same as class name in test_doc.py***
    "tag":  ""                  ***BETA: if you want to tag your strategy set some string for example `v2.0.0`***
  },
  "backtest": {
    "enabled": true,            ***true|false / if you want to optimize your strategy set it to false***
    "instruments": [            ***you can add multiple instruments to backtest together***
      {
        "plot": {
          "conservative_plot": true,  ***to show output chart.html without indicators***
          "include_candle": true,
          "include_volume": true,
          "include_legend": true,
          "indicators": [
            {
              "indicator_name": "ichimoku",
              "indicator_id": "",
              "sub_indicator_names": [
                "tenkan_sen"
              ]
            }, 
            {
              "indicator_name": "macd",
              "indicator_id": "1"
            }
          ]
        },
        "sub_indicators_for_debug": [  ***each indicator may have multiple lines***
          "ichimoku_sym_tenkan_sen",   ***add the name of indicator and its sub_line to together and set a `_` between them***
          "ichimoku_sym_kijun_sen",    
          "rsi"                        ***if your indicator only have one line in it only  repeat the name of indicator***
        ],
        "market": "tse",               ***"tse"|"crypto"|"forex"
        "exchange": "tse",             ***"tse"|"kucoin"|"oanda"
        "symbols": [                   ***datas[0]***
            "اهرم"
        ],
        "additional_datas": [          ***multiple data feeds to your strategy***
            {
                "market": "tse",       ***datas[1]***
                "exchange": "tse",    
                "symbol": "شاخص کل6"
            }
        ],
        "start_date": "2010-01-01 00:00:00",
        "end_date": "2023-05-01 00:00:00",
        "timeframes": [                ***important note: we do not support multiple elements for this list***
            [
                "1d"                   ***you can specify other timeframes too, for example '1h' '1m' '15m' '1w' etc.***
            ]
        ],
        "data_format": "mid",          
        "starting_capital": 10000,      
        "leverage": 1,
        "commission": 0.0012,          ***not in percentage but in fraction***
        "instant_profit_withdraw": false,   ***optional***
        "open_position_on_close_price": true,
        "close_position_on_close_price": true,
        "analyzers": [                 
          "trade_list",                
          "returns",
          "chart",
          "tearsheet",
          "open_positions"
        ],
        "observers": []
      }
    ]
  },
  "parameter_optimization": {
    "enabled": false,                       ***true|false***
    "search_space": {                      
      "rsi_period": {                       ***incremental and quantitative params***
        "type": "int",
        "min": 5,
        "max": 7,
        "step": 1                           ***[5, 6, 7]***
      }
    },
    "optimization_method": "GridSampler",   
    "objectives": ["sharpe"],               ***optimize based on***
    "directions": ["maximize"],             ***optimization approach for the objective(s)***
    "callbacks": {},
    "n_trials": 10,                         ***after n_trials return***
    "stats": [                              ***report the following***
      "win_rate",
      "balance_drawdown",
      "total_profit"
    ],
    "save_results_per_iteration": true,          
    "plots": ["optimization_history", "slice", "parallel_coordinate", "contour", "param_importances"]
  },
  "money_management": {            ***BETA***
    "enabled": false
  },
  "sl_tp_optimization": {          ***BETA***
    "enabled": false
  },
  "results": {
    "backtest": [],
    "parameter_optimization": [],  
    "money_management": [],
    "sl_tp_optimization": []
  }
}
```

# Backtest
***
  The "backtest" section of this configuration file,
  enables users to backtest their trading strategy 
  based on their desired details.

## Configuration Items

### enabled
  A boolean value that determines whether backtest is turned on or off.

### without_legend 
  If for any reason you want to hide all legends on the top-left of the chart you
  can add `"include_legend": true` under `"backtest.plot"` key.

### instruments
  Each unit of the backtest is referred to as an instrument, 
  and it contains information regarding data types, backtest configurations, 
  and other settings, which are elaborated on below.

#### market
  Specifies the market of the data to be used for conducting the backtest.

##### Available markets
  * **forex**
  * **crypto**
  * **tse** <details>Refers to Iran's stock market</details>

#### exchange
  Specifies the exchange of the data to be used for conducting the backtest.

##### Available exchanges
  * **forex**
    * oanda
  * **crypto**
    * kucoin
  * **tse** <details>Refers to Iran's stock market</details>
    * tse


#### symbols
  You can specify the symbols to be used in the backtest using this setting.

  You can give multiple symbols of one exchange in a config like below:

  ```
  "symbols": [
          "شاخص کل",
          "اهرم"
        ],
  ```

  BETA -> The backtest will be performed on all the specified symbols using the
  pre-set parameters defined in this instrument part.


#### additional_datas
  This field is used to specify additional data 
  to be included with the primary data feed while backtesting. 
  
  Please ensure that the format of this field is correct.

#### start_date
  Specifies the starting date for the backtest.
  correct format: YYYY-MM-DD HH:MM:SS (2022-01-01 00:00:00)

#### end_date
  Specifies the ending date for the backtest.
  correct format: YYYY-MM-DD HH:MM:SS (2023-01-01 00:00:00)

#### timeframes
  Specifies the timeframes you want to use in your backtest.
  
  For instance, if you have a multi-timeframe strategy, such as 1 day and 4 hours, you should set this field to: ["1d", "4h"].

#### data_format
  Specifies the format of the data.

##### Available data_formats
  * **mid**

#### starting_capital
  Specifies the initial amount of cash to start the backtest with.

#### leverage
  Specifies the leverage amount.

#### commission
  Specifies the leverage amount.

  Please note that this value MUST NOT BE specified in percentage format (e.g., 0.12 should not be interpreted as 0.12%).

#### instant_profit_withdraw
  Specifies whether to withdraw profits after each trade that is closed, based on your **starting_capital**. 
  
  Note that if the sum of your current balance and the profit from a trade is greater than your **starting_capital**, the profit will be withdrawn after the trade is closed.
#### open_position_on_close_price
  A boolean value that determines whether trades should be opened and closed based on the closing price of a candle or not.

#### close_position_on_close_price
  BETA

#### analyzers
  Specifies the list of the analyzers you want to see in the results.

##### Available analyzers
  * **trade_list**
  * **returns**
  * **tearsheet**
  * **chart**
  * **open_positions**

  
#### observers
  BETA

## Backtest Essential Notes
  * You can use the following code to add More info to your chart 
    and trade_list 'self.buy(size=0.001, **MoreInfo="any string you want"**)'. 
    It works for self.sell too!

# Parameter Optimization
***
  The "parameter_optimization" section of this configuration file,
  enables users to optimize the performance of their trading strategy 
  by searching for the best combination of parameter values 
  within a predefined search space.

## Configuration Items

### enabled
  A boolean value that determines whether parameter optimization is turned on or off.

### search_space
  The "search_space" parameter is a dictionary that defines the search space for the parameters to be optimized. It contains key-value pairs, where **each key represents the name of a parameter to be optimized**, and each value represents the range of values that the parameter can take.
    
  For numeric parameters, such as "rsi_period" in the example, the value of the key should be a dictionary that contains the following keys:
      
    "type": the data type of the parameter (e.g., "int", "float")
    "min": the minimum value of the parameter
    "max": the maximum value of the parameter
    "step": the step size between values in the search space

  For categorical parameters, such as "l_close_pr" in the example, the value of the key should be a dictionary that contains the following keys:

    "type": the data type of the parameter (i.e., "categorical")
    "choices": a list of possible values that the parameter can take

### optimization_method
  The algorithm used to find good solutions for a problem given certain constraints and objectives. 
  
#### Available Methods
* **GridSampler**: <details>With Grid Sampler, the trials suggest all combinations of parameters in the given search space during the study.</details>
* **RandomSampler**: <details>The Random Sampler uses random sampling.</details>
* **TPESampler**: <details>The TPE(Tree-structured Parzen Estimator) Sampler employs a Bayesian optimization strategy to guide the search process. It builds a probability model of the objective function based on the observed hyperparameter configurations and corresponding evaluation results. This model is then used to suggest new hyperparameter configurations to evaluate, with the goal of maximizing the objective function.</details>
* **CmaEsSampler**: <details>The CMA-ES(Covariance Matrix Adaptation Evolution Strategy) Sampler is inspired by natural evolution processes and uses a combination of exploration and exploitation to search for optimal hyperparameter configurations. It maintains a population of candidate solutions, called individuals, and iteratively updates their distributions based on their fitness evaluations.</details>


#### Essential Tips
1. Avoid using "GridSampler" if your search space is huge, as it may result in an impractical number of trials.
2Avoid using "CmaEsSampler" if you have categorical parameter(s) in your search space.

### objectives
  The objective function takes as input one or more variables, which are referred to as decision variables, and outputs a single value that represents the objective or goal of the optimization problem.
    
#### Available Functions
* **sharpe**: <details>The Sharpe ratio compares the return of an investment with its risk. <br> _Sharpe Ratio_= ```\frac{R_p - R_f}{\sigma_p}``` <br> Where: <br> R_p: return of portfolio <br> R_f: risk-free rate <br> \sigma_p: standard deviation of the portfolio's excess return </details>
* **balance_drawdown**: <details>The balance drawdown refers to the decrease in the balance of a trading account from its peak value to its lowest value over a specified period of time. This decline is expressed as a percentage amount and can provide insight into the risk and volatility of a trading strategy.</details>

#### Essential Tips
1. Even if there is only one objective function, it should be listed in a format that uses a list.
2. If there are multiple objective functions, this indicates that the optimization process is multi-objective.

### directions
  The directions of the objective function(s) refer to the optimization approaches for those objectives.

#### Essential Tips
1. The number of **directions** in the list must match the number of **objectives**.
2. The direction must be specified as either "maximize" or "minimize".

### callbacks
    BETA

### n_trials
  The number of times the objective function is evaluated while searching for the optimal set of hyperparameters.

#### Essential Tips
1. Avoid setting a large value for "n_trials," as it may result in an impractical number of trials.

### stats
Following each iteration, the specified statistics and parameter values will be recorded in a CSV file.

#### Available Stats
* **win_rate**
* **balance_drawdown**
* **equity_drawdown**
* **total_profit**
* **number_of_trades**

### save_results_per_iteration
  A boolean value that determines whether saving results per iteration is turned on or off.

#### Essential Tips
1. Avoid setting "save_results_per_iteration" to true if you have a lot of trials, as it may result in significant occupation of storage space.

### plots
  At the end of the parameter optimization process, you will have these collection of plots that contain valuable information about the entire process.

#### Available Plots
* **optimization_history**: <details>Plot optimization history of all trials in a study.</details>
* **parallel_coordinate**: <details>Plot the high-dimensional parameter relationships in a study. Note that, if a parameter contains missing values, a trial with missing values is not plotted.</details>
* **slice**: <details>Plot the parameter relationship as slice plot in a study. Note that, if a parameter contains missing values, a trial with missing values is not plotted.</details>
* **contour**: <details>Plot the parameter relationship as contour plot in a study. Note that, if a parameter contains missing values, a trial with missing values is not plotted.</details>
* **param_importances**: <details>Plot hyperparameter importances.</details>

#### Essential Tips
1. In the case of multiple objective functions, only an automatic Pareto plot will be generated, and any other plots will be disregarded.

### `/error`
For checking errors raised in your strategy
Input strategy name 
Here is *test_doc*

#### some common errors
* `config.json` has some problems
* You did not download approperiate data
* syntax error in your strategy 

### `/result/<file_name>`
Get the complete results
Input {strategy name}_{iteration}
for example *test_doc_1*


## Data Service
`ds.infra.goldscanco.com/docs`
### `/update`

* download data
* returns {"task_id": __id__} 
    
### You can give the task_id to `/task/{task_id}` and check the state of the task.

#### forex-oanda
```
{
  "market": "forex",
  "exchange": "oanda",
  "pair": "XAU_USD",
  "time_frame": "1M",
  "start": "2020-02-02 22:00:00",
  "end": "2023-02-02 22:00:00",
  "args": {
    "price_mode": "M" | "B" | "A",       ***Mid or Bid or Ask***
    "smooth": true|false                 
  },
  "username": "sajad",
  "password": "123456789"
}
```

#### tse-tse
Only support `1d`

##### for tickers|index you should set args differently
```
{
  "market": "tse",
  "exchange": "tse",
  "pair": "اهرم",
  "time_frame": "1d",
  "start": "2020-02-02 22:00:00",
  "end": "2023-02-02 22:00:00",
  "args": {"is_index": false|true},
  "username": "sajad",
  "password": "123456789"
}
```

#### crypto-kucoin
```
{
  "market": "crypto",
  "exchange": "kucoin",
  "pair": "BTC_USDT",
  "time_frame": "1m",
  "start": "2020-02-02 22:00:00",
  "end": "2023-02-02 22:00:00",
  "args": {},
  "username": "sajad",
  "password": "123456789"
}
```
### `/batch-update`
* update pairs in batch mode
* you need to prepare a json like below for your desired pairs to use batch_update.
```
[
    {
        "market": "tse",
        "exchange": "tse",
        "pair": "اهرم",
        "time_frames": ["1d"]
    },
    {
        "market": "crypto",
        "exchange": "kucoin",
        "pair": "BTC_USDT",
        "time_frames": ["1h", "2h", "1d"]
    }
]
```
* fill the username, password and the start field like other parts of the data service.
* return `json` of task ids

### `/data`

* get downloaded data from server
* return `json` if everything goes well
    `{"history": []}`

# Money Management
Implement and Use different money management methods.

## How to use?
### Initialization
```python
    def __init__(self, params=None) -> None:
        # ------------------other stuff here--------------------- #
        self.mm_algo_builder = self.get_MM_obj()
        self.mm_algo_builder.init_volume = 10000
        self.mm_algo_builder.volume_step = 100
        self.mm_algo_builder.money_management_method = "offensive"
        self.mm_algo = self.mm_algo_builder.get_algorithm("dealenbert")
        # ------------------other stuff here--------------------- #
```
### In the `next` method
```python
    def next(self):
        # ------------------other stuff here--------------------- #
        vol = self.mm_algo.suggest_volume()
        self.log(txt=str(vol))
        # ------------------other stuff here--------------------- #

```
## Types of Money Management 
### donguy_simple
<details>
<summary>Inputs</summary>
 
 * self.init_volume (float),
 * self.money_management_method (str) (offensive|defensive),
 * self.volume_step (float),

 </details>
 
<details>
 
<summary>How does it work?</summary>

The `donguy_simple` method is another money management technique tailored for trading strategies. Like Dealenbert, it adapts trading volume using past trade outcomes and the chosen money management mode. Under the "offensive" mode, a positive trade profit prompts volume growth by a predefined step. Similarly, in the "defensive" mode, a negative profit from the last trade results in volume increase.

</details>


### dealenbert

<details>
<summary>Inputs</summary>
 
 * self.init_volume (float),
 * self.money_management_method (str) (offensive|defensive),
 * self.volume_step (float),

</details>

<details>
 
<summary>How does it work?</summary>

The `dealenbert` method represents a money management approach for trading strategies. It adjusts trading volume based on previous trade performance and the selected money management method. In the "offensive" mode, if the last trade yielded positive profit, the method suggests increasing the trading volume by a specified step. Conversely, in the "defensive" mode, a negative profit from the last trade results in volume increase, while positive profit leads to volume reduction.

</details>

## creating new money management methods

<details>
<summary>Instructions</summary>

Just make a method inside the strategy with the following signature: `[self] -> float`.
If you need to set any configuration for this method, use `self.sample_config = 1019` inside the `__init__`.

You can implement any desired logic within the method. After implementation, inform the infrastructure support team to add the method to the infrastructure.
</details>

# FAQ
## Why is server slow?
* Which endpoint is slow? If you are downloading data(`:8000/update`) or strategy result(`:5001/result/{file_name}`) it is normal to be slow due to network limitations or/and size of data or result.
* If you are optimizing something(`:5001/upload`) it is normal to be slow, too. Assume each backtest takes 2 secs and you can calculate the approximate time you need to wait. 


# Report Problem
Problem $\rightarrow$ Inform us in Slack $\rightarrow$ Ticket in Trello $\rightarrow$ Happy :)
