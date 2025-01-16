# Import required libraries
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import dash
from dash import dcc, html
from dash.dependencies import Input, Output
import dash_bootstrap_components as dbc
import numpy as np
import os

# Initialize the app
app = dash.Dash(
    __name__, 
    external_stylesheets=[
        dbc.themes.FLATLY,
        'https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap'
    ]
)

# This is important for Render deployment
server = app.server

# Custom CSS
app.index_string = '''<!DOCTYPE html>
<html>
    <head>
        {%metas%}
        <title>{%title%}</title>
        {%favicon%}
        {%css%}
        <style>
            * {
                font-family: 'Bebas Neue', sans-serif;
            }
            .regular-text {
                font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            }
            .card-body p, .card-body text {
                font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            }
            .card {
                margin-bottom: 1rem;
            }
        </style>
    </head>
    <body>
        {%app_entry%}
        <footer>
            {%config%}
            {%scripts%}
            {%renderer%}
        </footer>
    </body>
</html>'''

# Monthly data from PDF
monthly_data = pd.DataFrame({
    'Month': ['January', 'February', 'March', 'April', 'May', 'June', 
             'July', 'August', 'September', 'October', 'November', 'December'],
    'Count': [11, 1311, 1690, 2103, 1748, 1453, 1748, 1857, 2176, 2290, 2356, 2388],
    'Volume': [132.00, 112560637.00, 144006772.00, 170239035.00, 90449754.00, 
               118670138.00, 74778406.00, 64775529.00, 89125884.00, 80648137.00, 
               79566345.00, 105387726.00],
    'Success_Rate': [54.55, 100, 100, 95.34, 100, 100, 100, 100, 100, 100, 100, 100],
})

# Transaction distribution data
transaction_distribution = pd.DataFrame({
    'Range': ['0-1000', '1001-5000', '5001-10000', '10000+'],
    'Count': [2505, 4469, 2559, 11598],
    'Percentage': [11.90, 21.10, 12.10, 54.90]
})

# Start App Layout
app.layout = dbc.Container([
    # Header
    dbc.Row([
        dbc.Col([
            html.Div([
                html.Img(
                    src='/assets/vngrd.PNG',
                    className='logo', 
                    style={'height': '150px', 'object-fit': 'contain'}
                )
            ], style={
                'display': 'flex', 
                'justifyContent': 'center', 
                'alignItems': 'center', 
                'padding': '40px', 
                'marginBottom': '30px', 
                'width': '100%'
            }),
            html.H1(
                "2024 Annual Business Transfer Analysis", 
                className="text-primary text-center mb-4",
                style={'letterSpacing': '2px'}
            )
        ])
    ]),

    # Key Metrics Cards
    dbc.Row([
        # Total Transactions Card
        dbc.Col([
            dbc.Card([
                dbc.CardBody([
                    html.H5("Total Annual Transactions", className="card-title text-center"),
                    html.H2(
                        f"{21131:,.0f}", 
                        className="text-primary text-center"
                    ),
                    html.P([
                        html.Span("Monthly Average: ", className="regular-text"),
                        html.Span(
                            f"{monthly_data['Count'].mean():,.0f}",
                            className="regular-text text-success"
                        )
                    ], className="text-center")
                ])
            ], className="shadow-sm")
        ]),
        
        # Total Volume Card
        dbc.Col([
            dbc.Card([
                dbc.CardBody([
                    html.H5("Total Volume (KES)", className="card-title text-center"),
                    html.H2(
                        f"{1130208495/1e9:.2f}B", 
                        className="text-primary text-center"
                    ),
                    html.P([
                        html.Span("Monthly Average: ", className="regular-text"),
                        html.Span(
                            f"KES {monthly_data['Volume'].mean()/1e6:.1f}M",
                            className="regular-text text-success"
                        )
                    ], className="text-center")
                ])
            ], className="shadow-sm")
        ]),
        
        # Average Amount Card
        dbc.Col([
            dbc.Card([
                dbc.CardBody([
                    html.H5("Average Amount (KES)", className="card-title text-center"),
                    html.H2(
                        f"{53485.80:,.2f}", 
                        className="text-primary text-center"
                    ),
                    html.P([
                        html.Span("Success Rate: ", className="regular-text"),
                        html.Span(
                            f"{99.51}%",
                            className="regular-text text-success"
                        )
                    ], className="text-center")
                ])
            ], className="shadow-sm")
        ])
    ], className="mb-4"),

    # Monthly Volume Trends
    dbc.Row([
        dbc.Col([
            dbc.Card([
                dbc.CardHeader("Monthly Transaction Analysis"),
                dbc.CardBody([
                    dcc.Graph(
                        figure=go.Figure(data=[
                            go.Bar(
                                name='Volume',
                                x=monthly_data['Month'],
                                y=monthly_data['Volume']/1e6,
                                marker_color='rgba(26, 118, 255, 0.8)',
                                yaxis='y'
                            ),
                            go.Scatter(
                                name='Success Rate',
                                x=monthly_data['Month'],
                                y=monthly_data['Success_Rate'],
                                mode='lines+markers',
                                marker=dict(
                                    size=8,
                                    color='rgba(255, 128, 0, 0.8)'
                                ),
                                line=dict(
                                    width=2,
                                    color='rgba(255, 128, 0, 0.8)'
                                ),
                                yaxis='y2'
                            )
                        ]).update_layout(
                            title='Monthly Volume and Success Rate Trends',
                            yaxis=dict(
                                title='Volume (KES Millions)',
                                titlefont=dict(color='rgba(26, 118, 255, 0.8)'),
                                tickfont=dict(color='rgba(26, 118, 255, 0.8)')
                            ),
                            yaxis2=dict(
                                title='Success Rate (%)',
                                titlefont=dict(color='rgba(255, 128, 0, 0.8)'),
                                tickfont=dict(color='rgba(255, 128, 0, 0.8)'),
                                overlaying='y',
                                side='right',
                                range=[0, 100]
                            ),
                            height=400,
                            margin=dict(l=50, r=50, t=50, b=30),
                            legend=dict(
                                orientation="h",
                                y=1.1,
                                x=0.5,
                                xanchor='center'
                            )
                        )
                    )
                ])
            ], className="shadow-sm")
        ], width=12)
    ], className="mb-4"),

    # Transaction Distribution
    dbc.Row([
        dbc.Col([
            dbc.Card([
                dbc.CardHeader("Transaction Distribution Analysis"),
                dbc.CardBody([
                    dcc.Graph(
                        figure=go.Figure(data=[
                            go.Bar(
                                name='Count',
                                x=transaction_distribution['Range'],
                                y=transaction_distribution['Count'],
                                marker_color='rgba(26, 118, 255, 0.8)',
                                yaxis='y'
                            ),
                            go.Scatter(
                                name='Percentage',
                                x=transaction_distribution['Range'],
                                y=transaction_distribution['Percentage'],
                                mode='lines+markers',
                                marker=dict(
                                    size=8,
                                    color='rgba(255, 128, 0, 0.8)'
                                ),
                                line=dict(
                                    width=2,
                                    color='rgba(255, 128, 0, 0.8)'
                                ),
                                yaxis='y2'
                            )
                        ]).update_layout(
                            title='Transaction Distribution by Amount Range',
                            yaxis=dict(
                                title='Number of Transactions',
                                titlefont=dict(color='rgba(26, 118, 255, 0.8)'),
                                tickfont=dict(color='rgba(26, 118, 255, 0.8)')
                            ),
                            yaxis2=dict(
                                title='Percentage (%)',
                                titlefont=dict(color='rgba(255, 128, 0, 0.8)'),
                                tickfont=dict(color='rgba(255, 128, 0, 0.8)'),
                                overlaying='y',
                                side='right'
                            ),
                            height=400,
                            margin=dict(l=50, r=50, t=50, b=30),
                            legend=dict(
                                orientation="h",
                                y=1.1,
                                x=0.5,
                                xanchor='center'
                            )
                        )
                    )
                ])
            ], className="shadow-sm")
        ], width=12)
    ], className="mb-4"),

], fluid=True, className="p-4")

# Run the app
if __name__ == '__main__':
    port = int(os.environ.get("PORT", 8080))
    app.run_server(debug=False, host='0.0.0.0', port=port)