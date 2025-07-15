//@version=5
indicator("Green Candle Alert", overlay=false)

// Detect green candle (close > open)
isGreenCandle = close > open

// Create alert condition
alertcondition(isGreenCandle, title="Green Candle Closed", message="A green candle has closed on {{ticker}} at {{close}}")