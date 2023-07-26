//@version=5
indicator("Custom Indicator", overlay=true)

// Function to calculate the average of an array of values
float calculateAverage(float[] arr) =>
    var totalSum = 0.0
    for value in arr
        totalSum += value
    totalSum / max(1, array.size(arr))

// Function to detect high-risk upward movement in 1-minute timeframe
string detectHighRiskUpward(float[] prices) =>
    if array.size(prices) < 2
        na
    else
        var currentPrice = prices[array.size(prices) - 1]
        var previousPrice = prices[array.size(prices) - 2]

        var priceChange = currentPrice - previousPrice
        var percentageChange = (priceChange / previousPrice) * 100

        if percentageChange > 2.0
            "High-risk upward movement detected in 1-minute timeframe."
        else
            "No high-risk upward movement detected in 1-minute timeframe."

// Function to detect high-risk downward movement in 1-minute timeframe
string detectHighRiskDownward(float[] prices) =>
    if array.size(prices) < 2
        na
    else
        var currentPrice = prices[array.size(prices) - 1]
        var previousPrice = prices[array.size(prices) - 2]

        var priceChange = previousPrice - currentPrice
        var percentageChange = (priceChange / previousPrice) * 100

        if percentageChange > 2.0
            "High-risk downward movement detected in 1-minute timeframe."
        else
            "No high-risk downward movement detected in 1-minute timeframe."

// Function to detect low-risk downward movement in 5-minute timeframe
string detectLowRisk(float[] prices) =>
    if array.size(prices) < 2
        na
    else
        var currentPrice = prices[array.size(prices) - 1]
        var previousPrice = prices[array.size(prices) - 2]

        var priceChange = previousPrice - currentPrice
        var percentageChange = (priceChange / previousPrice) * 100

        if percentageChange > 2.0
            "Low-risk downward movement detected in 5-minute timeframe."
        else
            "No low-risk downward movement detected in 5-minute timeframe."

// Function to perform Fibonacci analysis on prices
void fibonacciAnalysis(float[] prices) =>
    if array.size(prices) < 2
        na
    else
        var highestPrice = ta.highest(prices)
        var lowestPrice = ta.lowest(prices)

        // Calculate Fibonacci levels
        var fib_0 = highestPrice
        var fib_0_236 = highestPrice - (0.236 * (highestPrice - lowestPrice))
        var fib_0_382 = highestPrice - (0.382 * (highestPrice - lowestPrice))
        var fib_0_5 = highestPrice - (0.5 * (highestPrice - lowestPrice))
        var fib_0_618 = highestPrice - (0.618 * (highestPrice - lowestPrice))
        var fib_1 = lowestPrice

        // Plot Fibonacci levels on the chart
        plot(fib_0, "Fib 0", color=color.gray, linewidth=1)
        plot(fib_0_236, "Fib 0.236", color=color.gray, linewidth=1)
        plot(fib_0_382, "Fib 0.382", color=color.gray, linewidth=1)
        plot(fib_0_5, "Fib 0.5", color=color.gray, linewidth=1)
        plot(fib_0_618, "Fib 0.618", color=color.gray, linewidth=1)
        plot(fib_1, "Fib 1", color=color.gray, linewidth=1)

// Declare and initialize variables
var float[] oneMinutePrices = array.new_float(0)
var float[] fiveMinutePrices = array.new_float(0)

// Function to get prices for the specified symbol and timeframe
void getPrices(string symbol, string timeframe, var float[] priceArray) {
    var price = request.security(symbol, timeframe, close);
    array.push(priceArray, price);
}

// Main function to analyze prices
void analyze() => {
    // Get all available symbols in TradingView
    var availableSymbols = request.security_list("", "STK")

    // Analyze all crypto prices for 1-minute and 5-minute timeframes
    for symbol in availableSymbols {
        array.clear(oneMinutePrices)
        array.clear(fiveMinutePrices)

        getPrices(symbol, "1", oneMinutePrices)
        getPrices(symbol, "5", fiveMinutePrices)

        // Analyze 1-minute timeframe
        for i = 0 to array.size(oneMinutePrices) - 1 {
            if i > 0 {
                "\n"
            }
            "Analyzing " + tostring(symbol) + " (1-minute timeframe):"
            var prices = array.slice(oneMinutePrices, i, i + 1)
            detectHighRiskUpward(prices)
            detectHighRiskDownward(prices)
            fibonacciAnalysis(prices)
        }

        // Analyze 5-minute timeframe
        for j = 0 to array.size(fiveMinutePrices) - 1 {
            "Analyzing " + tostring(symbol) + " (5-minute timeframe):"
            var prices = array.slice(fiveMinutePrices, j, j + 1)
            detectLowRisk(prices)
        }
    }
}

// Call the main analyze function
analyze()

// Example: Moving average
var length = input(14, "MA Length")
var source = input(close, "Source")
var smaValue = ta.sma(source, length)

// Plot the indicators on the chart
plot(smaValue, "SMA", color=color.blue)

// Plot a line showing the average of the last 5 SMA values
var smaValuesArray = array.new_float()
array.push(smaValuesArray, smaValue)
var averageSMA = calculateAverage(smaValuesArray)
plot(averageSMA, "Average SMA", color=color.red)

// Continue with your other indicators and calculations...
