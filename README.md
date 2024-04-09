//@version=5
strategy(title='EMA RSI 布林带策略', shorttitle='EMA RSI BB', overlay=true)

// 输入参数
emaLength = input.int(5, title="EMA 长度")
atrLength = input.int(5, title="ATR 长度")
bbLength = input.int(3, title="布林带长度")
bbStdDev = input.float(1, title="布林带标准差")
atrMultiplier = input.float(4.5, title="ATR 停损倍数")

// 获取不同指定时间段的数据

//Session Asia
show_sesasia = input(true, 'Asia Session', group = 'Asia Session', tooltip = 'Sydney + Tokyo')
sesasia_ses = input.session('2000-0001', '', inline = 'sesasia', group = 'Asia Session')
sesasia_color = input.color(#8bbcfc, 'Color', inline = 'sesasia', group = 'Asia Session')
sesasia_text = 'Asia'

//Session Sydney
show_sesSydney = input(true, 'Sydney Session', group = 'Sydney Session') 
sesSydney_ses = input.session('0300-0401', '', inline = 'sesSydney', group = 'Sydney Session')
sesSydney_color = input.color(#F0B884, 'Color', inline = 'sesSydney', group = 'Sydney Session')
sesSydney_text = 'London Silver Bullet'

//Session Tokyo
show_sesTokyo = input(true, 'Tokyo Session', group = 'Tokyo Session') 
sesTokyo_ses = input.session('0000-0600', '', inline = 'sesTokyo', group = 'Tokyo Session')
sesTokyo_color = input.color(#0CC1C0, 'Color', inline = 'sesTokyo', group = 'Tokyo Session')
sesTokyo_text = 'Tokyo'

//Session Shanghai
show_sesShanghai = input(false, 'Shanghai Session', group = 'Shanghai Session') 
sesShanghai_ses = input.session('1000-1101', '', inline = 'sesShanghai', group = 'Shanghai Session')
sesShanghai_color = input.color(color.rgb(255, 16, 16, 50), 'Color', inline = 'sesShanghai', group = 'Shanghai Session')
sesShanghai_text = 'New York AM Silver Bullet'

//Session Europe
show_sesEuro = input(true, 'Europe Session',  group = 'Europe Session', tooltip = 'Frankfurt + London') 
sesEuro_ses = input.session('1400-1501', '', inline = 'sesEuro', group = 'Europe Session')
sesEuro_color = input.color(#BBE8B5, 'Color', inline = 'sesEuro', group = 'Europe Session')
sesEuro_text = 'New York PM Silver Bullet'

show_seslondon = input(true, 'London Session',  group = ' London Session') 
seslondon_ses = input.session('0200-0501', '', inline = 'seslondon', group = ' London Session')
seslondon_color = input.color(#ACBBE8, 'Color', inline = 'seslondon', group = ' London Session')
seslondon_text = 'London'

//Session NewYork
show_sesNewyork = input(true, 'Newyork Session',  group = 'Newyork Session') 
sesNewyork_ses = input.session('0830-1101', '', inline = 'sesNewyork', group = 'Newyork Session')
sesNewyork_color = input.color(#C5ACE8, 'Color', inline = 'sesNewyork', group = 'Newyork Session')
sesNewyork_text = 'New York AM'

show_sesNYSE = input(true, 'NYSE',  group = 'NYSE Session') 
sesNYSE_ses = input.session('1330-1601', '', inline = 'sesNYSE', group = 'NYSE Session')
sesNYSE_color = input.color(#87c2d4, 'Color', inline = 'sesNYSE', group = 'NYSE Session')
sesNYSE_text = 'New York PM'

//Info Table
Show_Info_Table = input(true, 'Info Table', group   = 'Table')


//zones
On_sesasia = math.sign(nz(time(timeframe.period, sesasia_ses, 'UTC-4')))
On_sesEuro = math.sign(nz(time(timeframe.period, sesEuro_ses, 'UTC-4')))
On_sesTokyo = math.sign(nz(time(timeframe.period, sesTokyo_ses, 'UTC-4')))
On_sesShanghai = math.sign(nz(time(timeframe.period, sesShanghai_ses, 'UTC-4')))
On_seslondon = math.sign(nz(time(timeframe.period, seslondon_ses, 'UTC-4')))
On_sesNewyork = math.sign(nz(time(timeframe.period, sesNewyork_ses, 'UTC-4')))
On_sesNYSE = math.sign(nz(time(timeframe.period, sesNYSE_ses, 'UTC-4')))
On_sesSydney = math.sign(nz(time(timeframe.period, sesSydney_ses, 'UTC-4')))



//High & Low Session Detector 
LowHighSessionDetector(On_Session, Color_Session, Text_Session) =>
    var int Bar = 0
    var float High = 0.0 
    var float Low = 0.0
    var box BoX = na
    var label LabeL = na 
    if  (On_Session[1] == 0 and On_Session == 1)
        Bar := bar_index
        High := high
        Low := low
    else if (On_Session[1] == 1 and On_Session == 1)
        High := math.max(high , High) 
        Low :=   math.min(low , Low)
    else if On_Session == 0
        High := 0.0 
        Low := 0.0
        Bar := 0

    if On_Session > On_Session[1]
        BoX := box.new(bar_index,High, bar_index , Low, bgcolor = color.new(Color_Session, 85),
         border_color = color.rgb(34, 101, 155),
         border_style = line.style_dotted)
        
        LabeL := label.new(Bar, High , text = Text_Session ,xloc = xloc.bar_index, yloc = yloc.price, size = size.small ,
         style = (Text_Session == 'Sydney' or Text_Session == 'Europe' ) ? label.style_label_lower_right  : 
         Text_Session == 'Asia' ?  label.style_label_down :  label.style_label_lower_left,
         textcolor = Color_Session, color = color.rgb(255, 255, 255, 100))
    if On_Session and On_Session == On_Session[1]
        box.set_top(BoX, High)
        box.set_bottom(BoX, Low)
        box.set_right(BoX, bar_index)
        label.set_x(LabeL,math.round(math.avg(Bar,bar_index)))
        label.set_y(LabeL, High)


    [High , Low ]
  

if show_sesasia 
    [HighA, LowA]  = LowHighSessionDetector(On_sesasia, sesasia_color,sesasia_text)

if show_sesSydney 
    [HighS, LowS]  = LowHighSessionDetector(On_sesSydney,sesSydney_color,sesSydney_text)

if show_sesTokyo 
    [HighT, LowT]  = LowHighSessionDetector(On_sesTokyo,sesTokyo_color,sesTokyo_text)

if show_sesShanghai 
    [HighSh, LowSh]  = LowHighSessionDetector(On_sesShanghai,sesShanghai_color,sesShanghai_text)

if show_sesEuro 
    [HighE, LowE]  = LowHighSessionDetector(On_sesEuro,sesEuro_color,sesEuro_text)

if show_seslondon 
    [HighL, LowL]  = LowHighSessionDetector(On_seslondon,seslondon_color,seslondon_text)

if show_sesNewyork 
    [HighN, LowN]  = LowHighSessionDetector(On_sesNewyork,sesNewyork_color,sesNewyork_text)

if show_sesNYSE
    [HighNY, LowNY]  = LowHighSessionDetector(On_sesNYSE,sesNYSE_color,sesNYSE_text)

//Info Table

var Info_Table = table.new(position.top_right, 4, 10, bgcolor = #283c74, border_color = #53609bcb, border_width = 1, 
 frame_color = #373a46 , frame_width = 1)

if Show_Info_Table
    table.cell(Info_Table, 0, 0, 'TradingFinder', text_color = color.white, text_size = size.small)
    table.merge_cells(Info_Table, 0, 0, 3, 0)
    table.cell(Info_Table, 0, 1, 'Session', text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 1, 1, 'Status', text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 2, 1, 'Start(UTC)', text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 3, 1, 'End(UTC)', text_color = color.white, text_size = size.small)

    table.cell(Info_Table, 0, 2, sesasia_text, text_color = sesasia_color, text_size = size.small)
    table.cell(Info_Table, 0, 3, sesSydney_text, text_color = sesSydney_color, text_size = size.small)
    table.cell(Info_Table, 0, 4, sesTokyo_text, text_color = sesTokyo_color, text_size = size.small)
    table.cell(Info_Table, 0, 5, sesShanghai_text, text_color = color.new(sesShanghai_color, 1), text_size = size.small)   
    table.cell(Info_Table, 0, 6, sesEuro_text, text_color = sesEuro_color, text_size = size.small)
    table.cell(Info_Table, 0, 7, seslondon_text, text_color = seslondon_color, text_size = size.small)
    table.cell(Info_Table, 0, 8, sesNewyork_text, text_color = sesNewyork_color, text_size = size.small)
    table.cell(Info_Table, 0, 9, sesNYSE_text, text_color = sesNYSE_color, text_size = size.small)

    
    
    table.cell(Info_Table, 1, 2, (On_sesasia ? 'Open' : 'Closed')
      , bgcolor = On_sesasia ? #56a3a6 : #db504a
      , text_color = color.rgb(0, 0, 0)
      , text_size = size.small)
    table.cell(Info_Table, 1, 3, (On_sesSydney ? 'Open' : 'Closed')
      , bgcolor = On_sesSydney ? #56a3a6 : #db504a
      , text_color = color.rgb(0, 0, 0)
      , text_size = size.small)
    table.cell(Info_Table, 1, 4, (On_sesTokyo ? 'Open' : 'Closed')
      , bgcolor = On_sesTokyo ? #56a3a6 : #db504a
      , text_color = color.rgb(0, 0, 0)
      , text_size = size.small)
    table.cell(Info_Table, 1, 5, (On_sesShanghai ? 'Open' : 'Closed')
      , bgcolor = On_sesShanghai ? #56a3a6 : #db504a
      , text_color = color.rgb(0, 0, 0)
      , text_size = size.small)
    table.cell(Info_Table, 1, 6, (On_sesEuro ? 'Open' : 'Closed')
      , bgcolor = On_sesEuro ? #56a3a6 : #db504a
      , text_color = color.rgb(0, 0, 0)
      , text_size = size.small)
    table.cell(Info_Table, 1, 7, (On_seslondon ? 'Open' : 'Closed')
      , bgcolor = On_seslondon ? #56a3a6 : #db504a
      , text_color = color.rgb(0, 0, 0)
      , text_size = size.small)
    table.cell(Info_Table, 1, 8, (On_sesNewyork ? 'Open' : 'Closed')
      , bgcolor = On_sesNewyork ? #56a3a6 : #db504a
      , text_color = color.rgb(0, 0, 0)
      , text_size = size.small)
    table.cell(Info_Table, 1, 9, (On_sesNYSE ? 'Open' : 'Closed')
      , bgcolor = On_sesNYSE ? #56a3a6 : #db504a
      , text_color = color.rgb(0, 0, 0)
      , text_size = size.small)

    table.cell(Info_Table, 2, 2, '23:00' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 2, 3, '23:00' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 2, 4, '00:00' , text_color = color.white, text_size = size.small) 
    table.cell(Info_Table, 2, 5, '01:30' , text_color = color.white, text_size = size.small)  
    table.cell(Info_Table, 2, 6, '07:00' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 2, 7, '08:00' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 2, 8, '13:00' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 2, 9, '14:30' , text_color = color.white, text_size = size.small)

    table.cell(Info_Table, 3, 2, '06:00' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 3, 3, '05:00' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 3, 4, '06:00' , text_color = color.white, text_size = size.small) 
    table.cell(Info_Table, 3, 5, '06:57' , text_color = color.white, text_size = size.small)  
    table.cell(Info_Table, 3, 6, '16:30' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 3, 7, '16:30' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 3, 8, '22:00' , text_color = color.white, text_size = size.small)
    table.cell(Info_Table, 3, 9, '22:00' , text_color = color.white, text_size = size.small)



smoothK1 = input.int(3, minval=1, title='SmoothK1')
smoothD1 = input.int(3, minval=1, title='SmoothD1')
lengthRSI1 = input.int(9, minval=1, title='LengthRSI1')
lengthStoch1 = input.int(9, minval=1, title='LengthStoch1')
uselog1 = input(true, title='Log1')
srcIn1 = input(close, title='Source1')
showdivs1 = input(true, title='显示背离1')
showhidden1 = input(false, title='显示隐藏背离1')
showchan1 = input(false, title='显示背离通道1')

// 指标计算
ema = ta.ema(close, emaLength)
atr = ta.atr(atrLength)
bbUpper = ta.sma(close, bbLength) + bbStdDev * ta.stdev(close, bbLength)
bbLower = ta.sma(close, bbLength) - bbStdDev * ta.stdev(close, bbLength)

src1 = uselog1 ? math.log(srcIn1) : srcIn1
rsi1 = ta.rsi(src1, lengthRSI1)
kk1 = ta.sma(ta.stoch(rsi1, rsi1, rsi1, lengthStoch1), smoothK1)
d1 = ta.sma(kk1, smoothD1)
hm1 = input(false, title='使用 K 和 D1 的平均值')
avg_1 = math.avg(kk1, rsi1)
k1 = hm1 ? rsi1 : kk1

// RSI 背离逻辑
f_top_fractal1(_src) => _src[4] < _src[2] and _src[3] < _src[2] and _src[2] > _src[1] and _src[2] > _src[0]
f_bot_fractal1(_src) => _src[4] > _src[2] and _src[3] > _src[2] and _src[2] < _src[1] and _src[2] < _src[0]
f_fractalize1(_src) => f_bot_fractal1__1 = f_bot_fractal1(_src), f_top_fractal1(_src) ? 1 : f_bot_fractal1__1 ? -1 : 0

fractal_top1 = f_fractalize1(k1) > 0 ? k1[2] : na
fractal_bot1 = f_fractalize1(k1) < 0 ? k1[2] : na

high_prev1 = ta.valuewhen(fractal_top1, k1[2], 0)[2]
high_price1 = ta.valuewhen(fractal_top1, high[2], 0)[2]
low_prev1 = ta.valuewhen(fractal_bot1, k1[2], 0)[2]
low_price1 = ta.valuewhen(fractal_bot1, low[2], 0)[2]

regular_bearish_div1 = fractal_top1 and high[2] > high_price1 and k1[2] < high_prev1
hidden_bearish_div1 = fractal_top1 and high[2] < high_price1 and k1[2] > high_prev1
regular_bullish_div1 = fractal_bot1 and low[2] < low_price1 and k1[2] > low_prev1
hidden_bullish_div1 = fractal_bot1 and low[2] > low_price1 and k1[2] < low_prev1

// 交易进入条件
//longCondition = close < ema and regular_bullish_div1
//shortCondition = close > ema and regular_bearish_div1
// 交易进入条件修改为布林带条件
Status_on = On_sesNYSE or On_sesNewyork or On_seslondon

longCondition = close < bbLower and hidden_bullish_div1 and Status_on
shortCondition = close > bbUpper and hidden_bearish_div1 and Status_on
longCondition1 = close < bbLower and regular_bullish_div1 and Status_on
shortCondition1 = close > bbUpper and regular_bearish_div1 and Status_on

// 执行交易
if (longCondition)
    strategy.entry("Long", strategy.long)
if (shortCondition)
    strategy.entry("Short", strategy.short)
if (longCondition1)
    strategy.entry("Long", strategy.long)
if (shortCondition1)
    strategy.entry("Short", strategy.short)

// 停損條件
var float longStopLossPrice = na
var float shortStopLossPrice = na
var int longEntryBar = na
var int shortEntryBar = na

// 更新進入條件和停損價格
if (longCondition)
    longEntryBar := bar_index
    longStopLossPrice := close - atr * atrMultiplier // 设置基于ATR的初始停損價格
if (shortCondition)
    shortEntryBar := bar_index
    shortStopLossPrice := close + atr * atrMultiplier // 设置基于ATR的初始停損價格

// ATR移动停损逻辑
if (strategy.position_size > 0)
    longStopLossPrice := math.max(longStopLossPrice, close - atr * atrMultiplier) // 更新多头ATR移动停损价
    if (close < longStopLossPrice)
        strategy.close("Long")

if (strategy.position_size < 0)
    shortStopLossPrice := math.min(shortStopLossPrice, close + atr * atrMultiplier) // 更新空头ATR移动停损价
    if (close > shortStopLossPrice)
        strategy.close("Short")

if (strategy.position_size > 0)
    // 多頭停損條件
    if (close > bbUpper or (close < ema and regular_bearish_div1))
        strategy.close("Long")
    else if (ta.barssince(longCondition) > 5)
        strategy.close("Long")

if (strategy.position_size < 0)
    // 空頭停損條件
    if (close < bbLower or (close > ema and regular_bullish_div1))
        strategy.close("Short")
    else if (ta.barssince(shortCondition) > 5)
        strategy.close("Short")

// 繪圖
plot(ema, title="EMA", color=color.blue)
plot(bbUpper, "布林带上界", color=color.red)
plot(bbLower, "布林带下界", color=color.green)
