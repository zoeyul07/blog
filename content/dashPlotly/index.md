---
emoji: ''
title: '데이터 시각화: Dash & Plotly'
date: '2020-11-05 22:00:00'
author: seoyul
tags: dataVisualization
categories: dataVisualization
---

## Dash & Plotly
- 인터랙티브 시각화를 제공해주는 plotly와 웹 기반의 파이썬 어플리케이션의 프레임워크를 제공해주는 Dash를 이용하여 데이터를 시각화할 수 있다.

1) Dash Component
- Slider, check box, date picker, drop down 등을 사용할 수 있게 해준다.

2) plotly graph
- graph, chart, data validation, plot 처럼 data를 시각화 해준다. 

- map box, scatter plot, line chart, bar chart 등을 이용할 수 있다.

3) call back
- dash component와 plotly graph를 연결하여 interative하게 만들어준다.

```python
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go

import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

app = dash.Dash(__name__)

# import data
df = pd.read_csv("intro_bees.csv")
df = df.groupby(["State", "ANSI", "Affected by", "Year", "state_code"])[
    ["Pct of Colonies Impacted"]
].mean()
df.reset_index(inplace=True)
# print(df[:5])

# App layout -dash component 등 html 관련된 코드들은 모두 이 안에 들어가게된다.
app.layout = html.Div(
    [
        html.H1(
            "Web Application DashBoards with Dash",
            style={"text-align": "center", "font-family": "sans-serif"},
        ),
        dcc.Dropdown(
            id="select_year",
            options=[
                # label은 사용자가 보게되고 value csv 파일에서 가져오게 된다
                {"label": "2015", "value": 2015},
                {"label": "2016", "value": 2016},
                {"label": "2017", "value": 2017},
                {"label": "2018", "value": 2018},
            ],
            multi=False,
            value=2015,  # initial value
            style={"width": "40%", "font-family": "sans-serif"},
        ),
        html.Div(id="output_container", children=[]),
        html.Br(),
        dcc.Graph(id="my_bee_map", figure={}),
    ]
)

# callback -Connect the Plotly graphs with Dash Components
# callback 마다 function을 정의해야한다.
@app.callback(
    [
        Output(component_id="output_container", component_property="children"),
        Output(component_id="my_bee_map", component_property="figure"),
    ],
    [Input(component_id="select_year", component_property="value")],
)
# argument로 들어오는 것은 input과 직결된다.input이 하나라면 하나의 argument를 갖고, 두개라면 두개의 argument를 갖는다.
# argument는 component property를 나타낸다
def update_graph(option_selected):
    print(option_selected)
    print(type(option_selected))

    container = "The year chosen by user was: {}".format(option_selected)

    dff = df.copy()
    # user가 선택한 row만 갖고 온다.
    dff = dff[dff["Year"] == option_selected]
    # bees가 Varroa_mites에 의해 영향받은 row만 선택
    dff = dff[dff["Affected by"] == "Varroa_mites"]

    # Plotly Express
    fig = px.choropleth(
        # 위에서 filter된 data frame을 사용한다.
        data_frame=dff,
        locationmode="USA-states",
        locations="state_code",
        scope="usa",
        color="Pct of Colonies Impacted",
        color_continuous_scale=px.colors.sequential.YlGnBu,
        labels={"Pct of Colonies Impacted": "% of Bee Colonies"},
    )

    return container, fig


# runserver
if __name__ == "__main__":
    app.run_server(debug=True)

```

***
[Plotly](https://dailyheumsi.tistory.com/118)
[Create effective data visualizations of proportions](https://towardsdatascience.com/create-effective-data-visualizations-of-proportions-94b69ad34410)
[Build a web data dashboard in just minutes with Python](https://towardsdatascience.com/build-a-web-data-dashboard-in-just-minutes-with-python-d722076aee2b)
[Dash & Plotly](https://www.youtube.com/watch?v=tpkZqE_cJsE&list=PLCDERj-IUIFCaELQ2i7AwgD2M6Xvc4Slf&index=4)
[Introduction to Dash Plotly - Data Visualization in Python](https://www.youtube.com/watch?v=hSPmj7mK6ng&t=1564s)


```toc

```
