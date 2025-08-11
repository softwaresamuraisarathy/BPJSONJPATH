//@version=5
indicator("Big Candle Breakout", overlay=true)

// Input for big candle threshold (in %)
bigCandleThreshold = input.float(1.0, "Big Candle % Threshold", minval=0.1, step=0.1)

// Function to check if candle is big
isBigCandle() =>
    candleRange = math.abs(high - low)
    bodyRange = math.abs(close - open)
    avgRange = ta.sma(candleRange, 14)  // Using 14-period SMA for comparison
    bodyRange >= avgRange * (bigCandleThreshold / 100)

// Arrays to store last 5 big candles' highs and lows
var float[] bigCandleHighs = array.new_float(5, na)
var float[] bigCandleLows = array.new_float(5, na)

// Check if current candle is big
bigCandle = isBigCandle()

// Store big candle data
if bigCandle
    array.unshift(bigCandleHighs, high)
    array.unshift(bigCandleLows, low)
    // Keep only last 5
    if array.size(bigCandleHighs) > 5
        array.pop(bigCandleHighs)
        array.pop(bigCandleLows)

// Check if current close breaks any big candle high/low
breakoutUp = false
breakoutDown = false
for i = 0 to array.size(bigCandleHighs) - 1
    if not na(array.get(bigCandleHighs, i))
        if close > array.get(bigCandleHighs, i)
            breakoutUp := true
        if close < array.get(bigCandleLows, i)
            breakoutDown := true

// Plot big candles
plotshape(bigCandle, style=shape.triangleup, location=location.belowbar, color=color.yellow, size=size.small, title="Big Candle")

// Plot breakout signals
plotshape(breakoutUp, style=shape.triangleup, location=location.abovebar, color=color.green, size=size.small, title="Breakout Up")
plotshape(breakoutDown, style=shape.triangledown, location=location.belowbar, color=color.red, size=size.small, title="Breakout Down")

// Optional: Plot lines for remembered big candle levels
for i = 0 to array.size(bigCandleHighs) - 1
    if not na(array.get(bigCandleHighs, i))
        hline(price=array.get(bigCandleHighs, i), linestyle=hline.style_dashed, color=color.blue, title="Big Candle High")
        hline(price=array.get(bigCandleLows, i), linestyle=hline.style_dashed, color=color.blue, title="Big Candle Low")