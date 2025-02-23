# Python股票量化回测（学习版）
使用pandas和numpy库读取CSV文件并进行移动平均线策略量化回测的代码
# 这段代码通过 pandas 和 numpy 库读取 CSV 文件的数据并计算移动平均线（MA5、MA10 和 MA20）。
# 当 MA5 大于 MA10 并且 MA10 大于 MA20 时买入，当 MA5 小于 MA10 并且 MA10 小于 MA20 时卖出。
# 回测期间的买卖点日期、最终盈利资金、收益率，并生成收益走势图。
# 可以转Jupyter Notebook格式

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# 读取CSV文件
data = pd.read_csv('C:/Users/1/Desktop/603993历史数据1.csv', parse_dates=['Date'], index_col='Date')
#CSV股票走势文件可以私聊店主帮爬取下载

# 确保日期索引是单调递增的
data = data.sort_index()

# 计算移动平均线
data['MA5'] = data['Close'].rolling(window=5).mean()
data['MA10'] = data['Close'].rolling(window=10).mean()
data['MA20'] = data['Close'].rolling(window=20).mean()

# 设置回测参数
initial_cash = 100000
cash = initial_cash
shares = 0
commission = 0.0001
start_date = '2020-01-01'
end_date = '2024-10-11'

# 检查并处理缺失的日期
all_dates = pd.date_range(start=start_date, end=end_date)
data = data.reindex(all_dates).fillna(method='ffill')

# 过滤回测日期范围
data = data.loc[start_date:end_date]

buy_dates = []
sell_dates = []
portfolio_values = []

# 回测策略
for i in range(len(data)):
    if i < 20:
        portfolio_values.append(initial_cash)  # 在前20天保持初始资金
        continue
    if data['MA5'][i] > data['MA10'][i] > data['MA20'][i]:
        if cash > 0:
            shares = cash / data['Close'][i] * (1 - commission)
            cash = 0
            buy_dates.append(data.index[i])
    elif data['MA5'][i] < data['MA10'][i] < data['MA20'][i]:
        if shares > 0:
            cash = shares * data['Close'][i] * (1 - commission)
            shares = 0
            sell_dates.append(data.index[i])
    portfolio_values.append(cash + shares * data['Close'][i])

# 计算收益率
total_value = cash + shares * data['Close'][-1] * (1 - commission)
returns = (total_value - initial_cash) / initial_cash

# 计算夏普比率
returns_series = pd.Series(portfolio_values).pct_change().fillna(0)
sharpe_ratio = np.sqrt(252) * returns_series.mean() / returns_series.std()

# 计算年化收益率
annualized_return = (total_value / initial_cash) ** (1 / (len(data) / 252)) - 1

# 打印结果
# 也可增加参数结果。。。
print("最终资金：", total_value)
print("收益率：", returns)
print("夏普比率：", sharpe_ratio)
print("年化收益率：", annualized_return)

# 绘制收益走势图
plt.figure(figsize=(14, 7))
plt.plot(data.index, data['Close'], label='Close Price', color='black')
plt.plot(data.index, data['MA5'], label='MA5', color='blue')
plt.plot(data.index, data['MA10'], label='MA10', color='red')
plt.plot(data.index, data['MA20'], label='MA20', color='green')
plt.scatter(buy_dates, data.loc[buy_dates]['Close'], marker='^', color='g')
plt.scatter(sell_dates, data.loc[sell_dates]['Close'], marker='v', color='r')
plt.legend()
plt.title('Moving Average Crossover Strategy')
plt.xlabel('Date')
plt.ylabel('Price')
plt.grid(True)
plt.show()

# 绘制资金变化图
plt.figure(figsize=(30, 20))
plt.plot(data.index, portfolio_values, label='Portfolio Value', color='purple')
plt.legend()
plt.title('Portfolio Value Over Time')
plt.xlabel('Date')
plt.ylabel('Value')
plt.grid(True)
plt.show()
