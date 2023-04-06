# AI-CBOT
AI adding to a CBOT (Trading Robot)


"My Code in C#"

using System;
using System.Linq;
using cAlgo.API;
using cAlgo.API.Indicators;
using cAlgo.API.Internals;
using cAlgo.Indicators;

namespace cAlgo.Robots
{
    [Robot(TimeZone = TimeZones.UTC, AccessRights = AccessRights.None)]
    public class EMA : Robot
    {
        [Parameter("Robot ID", DefaultValue = "Harmonics")]
        public string RobotID { get; set; }

        [Parameter("Initial Lot Size", DefaultValue = 1, Group = "Anti-martingale Volume")]
        public double InitialLotSize { get; set; }

        [Parameter("Minimum Lot Size", DefaultValue = 0.1, Group = "Anti-martingale Volume")]
        public double MinLotSize { get; set; }

        [Parameter("Maximum Lot Size", DefaultValue = 100, Group = "Anti-martingale Volume")]
        public double MaxLotSize { get; set; }

        [Parameter("Total Net Profit Step", DefaultValue = 100, Group = "Anti-martingale Volume")]
        public double TotalNetProfitStep { get; set; }

        [Parameter("Lot Increase Step", DefaultValue = 0.4, Group = "Anti-martingale Volume")]
        public double LotIncreaseStep { get; set; }

        [Parameter("Stoploss Percentage", DefaultValue = 10, Group = "Risk Management")]
        public int Stoploss { get; set; }

        [Parameter("Take Profit in Pips")]
        public double TPPips { get; set; }
        
          [Parameter("Periods", DefaultValue = 20, Group = "Keltner")]
        public int Periods { get; set; }
        
        [Parameter("Periods ATR", DefaultValue = 20, Group = "Keltner")]
        public int PeriodsATR { get; set; }


        [Parameter("Multiplier", DefaultValue = 2, Group = "Keltner")]
        public double Multiplier1 { get; set; }
        
        
      [Parameter("Fast Period", DefaultValue = 12, Group = "Keltner")]
        public int FastPeriod { get; set; }

        [Parameter("Slow Period", DefaultValue = 26, Group = "Keltner")]
        public int SlowPeriod { get; set; }

        [Parameter("Signal Period", DefaultValue = 9, Group = "Keltner")]
        public int SignalPeriod { get; set; }

        [Parameter("Smoothing Factor", DefaultValue = 0.1, MinValue = 0.01, MaxValue = 1, Step = 0.01, Group = "Keltner")]
        public double SmoothingFactor { get; set; }

        [Parameter("Alternate Positions' Trade Direction", DefaultValue = true)]
        public bool AlternateTradeDirection { get; set; }

        [Parameter("Close Positions on Opposite EMA Cross Signal", DefaultValue = true)]
        public bool CloseOnOppositeCross { get; set; }

        [Parameter(DefaultValue = 10)]
        public int Period { get; set; }

        [Parameter(DefaultValue = 3.0)]
        public double Multiplier { get; set; }

        [Parameter(DefaultValue = MovingAverageType.Simple)]
        public MovingAverageType MaType { get; set; }

          [Parameter(DefaultValue = 20)]
        public int Periods1 { get; set; }

        [Parameter(DefaultValue = 50)]
        public int Periods2 { get; set; }
        
        [Parameter(DefaultValue = 0.0)]
       
        public double Spread1 { get; set; }

        private double volumeInUnits;

        double InitialEquity;
        private int _trendDirection;
        private int _lastTrendDirection;
private KeltnerMACDChannel Keltner;
        private VWAP Trend;
private bool _hasOpenedBuyTrade;
private bool _hasOpenedSellTrade;
        private TradeType? LastTradeType;
        private DateTime LastTradeOpenTime;

        protected override void OnStart()
        {
            InitialEquity = (Account.Equity/20);

            Trend = Indicators.GetIndicator<VWAP>(Periods1, Periods2);
            Keltner = Indicators.GetIndicator<KeltnerMACDChannel>(Periods, PeriodsATR, Multiplier1, FastPeriod,SlowPeriod, SignalPeriod,SmoothingFactor);
            
            LastTradeOpenTime = Bars.OpenTimes.Last(1);
            _trendDirection = 0;
    _lastTrendDirection = 0;
   _hasOpenedBuyTrade = false;
    _hasOpenedSellTrade = false;
        }

        protected override void OnTick()
        {
            double stopLossInPips = Stoploss; // the stop loss in pips
double maxEquityLossPercentage = 5; // the maximum equity loss percentage

double riskPerPip = (maxEquityLossPercentage / 100) * ((Account.Balance) / stopLossInPips); // calculate the risk per pip
volumeInUnits = Symbol.NormalizeVolumeInUnits(riskPerPip / Symbol.PipValue); // calculate the lot size based on the risk per pip

// set the volume to the smaller of its current value and the MaxLotSize variable

            
          

 
            
         
        }

        protected override void OnBar()
        {
         Symbol symbol = Symbol;
    double bidPrice = symbol.Bid;
    double askPrice = symbol.Ask;
    double spread = askPrice - bidPrice;
        
        
        
        double stopLossInPips = Stoploss; // the stop loss in pips
double maxEquityLossPercentage = 5;
double maxEquityProfitPercentage = 30; // the maximum equity profit percentage

// calculate the risk per pip
double riskPerPip = (maxEquityLossPercentage / 100) * ((Account.Balance) / stopLossInPips);
double profitPercentage = 20;
double profitAmount = (profitPercentage / 100) * (Account.Balance);
double profitPips = profitAmount / (riskPerPip * Symbol.PipValue);
          
        


// Check exit signals if close on opposite EMA entry signal activated.
if (CloseOnOppositeCross)
{
        
        if (_trendDirection != 0 && _trendDirection != _lastTrendDirection)
    {
        // close any open trades
        foreach (var position in Positions)
        {
            if (position.SymbolName == SymbolName && position.TradeType == TradeType.Buy && _trendDirection < 0)
            {
                ClosePosition(position);
         
            }
            else if (position.SymbolName == SymbolName && position.TradeType == TradeType.Sell && _trendDirection > 0)
            {
                ClosePosition(position);
            }
        }

        // update the last trend direction
        _lastTrendDirection = _trendDirection;
    
}
}
        
    


 if (Trend.VWAPLine1.Last(0) > Trend.VWAPLine2.Last(0) && Trend.VWAPLine1.Last(1) <= Trend.VWAPLine2.Last(1))
    {
        _trendDirection = 1; // uptrend
    }
    else  if (Trend.VWAPLine1.Last(0) < Trend.VWAPLine2.Last(0) && Trend.VWAPLine1.Last(1) >= Trend.VWAPLine2.Last(1))
    {
        _trendDirection = -1; // downtrend
    }

// Check entry signals.


{
if (LastTradeType == TradeType.Sell || LastTradeType == null && Positions.Count <1)
{
    if (spread <= Spread1)
    if((Trend.VWAPLine1.Last(0) > Trend.VWAPLine2.Last(0) && Trend.VWAPLine1.Last(1) <= Trend.VWAPLine2.Last(1)) && !_hasOpenedBuyTrade)
    {
        // Execute market order.
        ExecuteMarketOrder(TradeType.Buy, SymbolName, volumeInUnits, RobotID, Stoploss, profitPips);
        // Save present bar's opening time to avoid opening any more trades on this bar.
        LastTradeOpenTime = Bars.OpenTimes.LastValue;
        
     
        _hasOpenedBuyTrade = true;
        _hasOpenedSellTrade = false;
            
     

            if (AlternateTradeDirection)
            {
                // If activated, save the trade direction of the market order, so that the next order opened by the robot is in the opposite direction.
                LastTradeType = TradeType.Buy;
            }
        }
    }

   if (LastTradeType == TradeType.Buy || LastTradeType == null && Positions.Count <1)
{
    if (spread <= Spread1)
    if(Trend.VWAPLine1.Last(0) < Trend.VWAPLine2.Last(0) && Trend.VWAPLine1.Last(1) >= Trend.VWAPLine2.Last(1) && !_hasOpenedSellTrade)
    {
        // Execute market order.
        ExecuteMarketOrder(TradeType.Sell, SymbolName, volumeInUnits, RobotID, Stoploss, profitPips);
        // Save present bar's opening time to avoid opening any more trades on this bar.
        LastTradeOpenTime = Bars.OpenTimes.LastValue;
        
     
        _hasOpenedBuyTrade = false;
        _hasOpenedSellTrade = true;

        if (AlternateTradeDirection)
        {
            // If activated, save the trade direction of the market order, so that the next order opened by the robot is in the opposite direction.
            LastTradeType = TradeType.Sell;
        }
    }

    }
    }
    }
    


protected override void OnStop()
{
    // Put your deinitialization logic here
}

private void UpdateTP()
        {
            double variableTP2 = Keltner.Lower.LastValue;
            double variableTP = Keltner.Upper.LastValue;


            foreach (Position myPosition in Positions.FindAll(RobotID))
            {
                // If the position has no TP, set TP to latest variable TP.
                if ((!myPosition.TakeProfit.HasValue))
                {
                    TradeResult result = myPosition.ModifyTakeProfitPrice(variableTP);

                    if (result.IsSuccessful)
                    {
                        Print(string.Format("Take profit modified to {0} for position with ID {1}.", result.Position.TakeProfit.Value, result.Position.Id));
                    }

                    else
                    {
                        Print(string.Format("Take profit could not be modified for position with ID {0}. Error: {1}.", myPosition.Id, result.Error));
                    }
                }

                // If the position has TP, check if latest variable TP value is better than the current TP.
                else
                {
                    if (myPosition.TradeType == TradeType.Buy)
                    {
                        // Update TP to a higher level.
                        TradeResult result = myPosition.ModifyTakeProfitPrice(variableTP);


                        if (result.IsSuccessful)
                        {
                            Print(string.Format("Take profit modified to {0} for position with ID {1}.", result.Position.TakeProfit.Value, result.Position.Id));
                        }

                        else
                        {
                            Print(string.Format("Take profit could not be modified for position with ID {0}. Error: {1}.", myPosition.Id, result.Error));
                        }
                    }

                    else if (myPosition.TradeType == TradeType.Sell)
                    {
                        // Update TP to lower level.
                        TradeResult result = myPosition.ModifyTakeProfitPrice(variableTP2);


                        if (result.IsSuccessful)
                        {
                            Print(string.Format("Take profit modified to {0} for position with ID {1}.", result.Position.TakeProfit.Value, result.Position.Id));
                        }

                        else
                        {
                            Print(string.Format("Take profit could not be modified for position with ID {0}. Error: {1}.", myPosition.Id, result.Error));
                        }
                    }
                }
            }
        }
        }
        }


