# Apartment-Price-Prediction---Rebuild

Apartment Price Predicition Rebuild 

### starts 2022-04-20
1. Pandas default output settings
```python
import pandas as pd
pd.set_option('display.max_row', 500)
pd.set_option('display.max_columns', 200)
pd.set_option('display.max_info_rows', 500)
pd.set_option('display.max_info_columns', 200)
```
2. Using the bokeh library I made earlier. [Referencㄷ](https://github.com/Fortuna3Co/University-field-training/blob/main/python_bokeh.ipynb])
```python
from bokeh.plotting import figure, show, output_notebook, output_file, save
from bokeh.models import LinearAxis, Range1d, ColumnDataSource, HoverTool, WheelZoomTool, Button, TextInput, CustomJS, CheckboxButtonGroup
from bokeh.embed import file_html
from bokeh.layouts import row, gridplot
from bokeh.palettes import Spectral11 as palette
from bokeh.io import curdoc, push_notebook
output_notebook()

def make_graphs(datas, y_axis, x_axis, title, x_axis_label, y_axis_label, size, alpha):
  plots = []
  # palette 최대 갯수
  max_color = 10

  if len(y_axis) > max_color:
    print("y축은 최대 11개 까지 가능합니다.")
    return None

  for i in range(len(datas)):
    curdoc().theme = "caliber"
    TOOLS = "pan,wheel_zoom,box_select,lasso_select,reset, save, hover"
    TOOLTIPS = [
      ("index", "$x"),
      ("y_value", "$y)"),
    ]

    p = figure(title=title, x_axis_label=x_axis_label, y_axis_label=y_axis_label, plot_width=1200, plot_height=900, tools=TOOLS, tooltips=TOOLTIPS)

    for j in range(0, len(y_axis)):
      # 색깔 지정을 위한 if, else
      if j == 0:
        p.extra_y_ranges = {y_axis[j] : Range1d(start=min(datas[i][y_axis[j]]), end=max(datas[i][y_axis[j]]))}
        p.circle(datas[i][x_axis], datas[i][y_axis[j]], y_range_name=y_axis[j], line_color=palette[j], size=size, alpha=alpha)

      else:
        p.extra_y_ranges[y_axis[j]] = Range1d(start=min(datas[i][y_axis[j]]), end=max(datas[i][y_axis[j]]))
        if j % 2 == 0:
          p.circle(datas[i][x_axis], datas[i][y_axis[j]], y_range_name=y_axis[j], line_color=palette[j], size=size, alpha=alpha)
          p.add_layout(LinearAxis(y_range_name=y_axis[j], axis_label=y_axis[j], axis_label_text_color=palette[j], axis_line_color=palette[j]), "right")
        else :
          p.circle(datas[i][x_axis], datas[i][y_axis[j]], y_range_name=y_axis[j], line_color=palette[-j], size=size, alpha=alpha)
          p.add_layout(LinearAxis(y_range_name=y_axis[j], axis_label=y_axis[j], axis_label_text_color=palette[-j], axis_line_color=palette[j]), "right")

    p.legend.orientation = "horizontal"
    p.legend.location = "top_right"
    p.legend.click_policy = "hide"
    p.background_fill_color="#f0f0f0"
    p.grid.grid_line_color="#f5f5f5"
    p.toolbar.active_scroll = p.select_one(WheelZoomTool)

    plots.append(p)
  return plots
```
3. Identify each data type to differentiate between numeric and categorical data. 
``` python
train.info()

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1933 entries, 0 to 1932
Data columns (total 41 columns):
 #   Column                       Dtype  
---  ------                       -----  
 0   Auction_key                  int64  
 1   Auction_class                object 
 2   Bid_class                    object 
 3   Claim_price                  float64
 4   Appraisal_company            object 
 5   Appraisal_date               object 
 6   Auction_count                int64  
 7   Auction_miscarriage_count    int64  
 8   Total_land_gross_area        float64
 9   Total_land_real_area         float64
 10  Total_land_auction_area      float64
 11  Total_building_area          float64
 12  Total_building_auction_area  float64
 13  Total_appraisal_price        float64
 14  Minimum_sales_price          float64
 15  First_auction_date           object 
 16  Final_auction_date           object 
 17  Final_result                 object 
 18  Creditor                     object 
 19  addr_do                      object 
 20  addr_si                      object 
 21  addr_dong                    object 
 22  addr_li                      object 
 23  addr_san                     object 
 24  addr_bunji1                  float64
 25  addr_bunji2                  float64
 26  addr_etc                     object 
 27  Apartment_usage              object 
 28  Preserve_regist_date         object 
 29  Total_floor                  int64  
 30  Current_floor                int64  
 31  Specific                     object 
 32  Share_auction_YorN           object 
 33  road_name                    object 
 34  road_bunji1                  float64
 35  road_bunji2                  float64
 36  Close_date                   object 
 37  Close_result                 object 
 38  point.y                      float64
 39  point.x                      float64
 40  Hammer_price                 float64
dtypes: float64(15), int64(5), object(21)
memory usage: 619.3+ KB
```

4. Check for missing values
```python
train.isnull().sum()

addr_li                        1910
addr_bunji2                    1044
Specific                       1869
road_bunji2                    1778
```

5.  Create graphs using Bokeh library. addr_li, addr_bunji2, Specific, road_bunji2 The column is excluded because there are missing values.
```python
# addr_li, addr_bunji2, Specific, road_bunji2 해당 컬럼은 결측치가 존재하므로 제외
train_numerical_columns = ['Auction_key', 'Claim_price', 'Auction_count',
       'Auction_miscarriage_count', 'Total_land_gross_area', 'Total_land_real_area',
       'Total_land_auction_area', 'Total_building_area', 'Total_building_auction_area',
       'Total_appraisal_price', 'Minimum_sales_price', 'addr_bunji1', 'Total_floor', 
       'Current_floor', 'road_bunji1', 'point.y', 'point.x']

plots = []
for t in train_numerical_columns:
       plots.append(make_graphs([train], [t], "Hammer_price", t, "Hammer_rice", t, 3, 0.4)[0])

plot = gridplot(plots, ncols=6, toolbar_location="right", width=400, height=300)
show(row(plot))
```
5-1. Using matplotlib (before)
![Matplot_Graph1](https://user-images.githubusercontent.com/78258412/164237130-12ff490b-1026-4d43-93f0-73101a71d527.png)

5-2. Using Bokeh Library (After)
![Bokeh_Graph1](https://user-images.githubusercontent.com/78258412/164237141-c8907bd6-5992-4d46-9044-612758665bde.PNG)



