from flask import Flask, render_template, request
import pandas as pd

app = Flask(__name__)

@app.route('/', methods=['GET', 'POST'])
def index():
    # Default input values
    inputs = {
        'revenue': 500_000_000,
        'controls_cost': 1_100_000,
        'user_count': 600_000,
        'monitoring_cost_per_user': 10,
        'sle_base': 6_000_000,
        'downtime_days': 5,
        'cost_per_day': 1_369_863,  # 500M / 365
        'aro_before': 0.30,
        'aro_after': 0.10,
        'maturity_modifier': 1.0,
        'maturity_level': 'Defined'
    }

    maturity_modifiers = {
        "Initial": 1.3,
        "Developing": 1.15,
        "Defined": 1.0,
        "Managed": 0.85,
        "Optimized": 0.7
    }

    if request.method == 'POST':
        # Update inputs from form
        for key in inputs:
            if key in request.form:
                val = request.form[key]
                if key in ['maturity_level']:
                    inputs[key] = val
                else:
                    try:
                        inputs[key] = float(val.replace(',', ''))
                    except ValueError:
                        inputs[key] = inputs[key]  # keep default

        inputs['maturity_modifier'] = maturity_modifiers.get(inputs['maturity_level'], 1.0)

    # Derived values
    user_breach_cost = inputs['user_count'] * inputs['monitoring_cost_per_user']
    downtime_cost = inputs['downtime_days'] * inputs['cost_per_day']
    sle = inputs['sle_base'] + user_breach_cost + downtime_cost
    aro_before_adj = inputs['aro_before'] * inputs['maturity_modifier']
    aro_after_adj = inputs['aro_after'] * inputs['maturity_modifier']

    ale_before = sle * aro_before_adj
    ale_after = sle * aro_after_adj
    risk_reduction = ale_before - ale_after

    roi = (risk_reduction / inputs['controls_cost']) if inputs['controls_cost'] > 0 else 0
    ale_before_pct = (ale_before / inputs['revenue']) * 100 if inputs['revenue'] else 0
    ale_after_pct = (ale_after / inputs['revenue']) * 100 if inputs['revenue'] else 0
    risk_red_pct = (risk_reduction / inputs['revenue']) * 100 if inputs['revenue'] else 0

    # Cost breakdown chart data
    cost_data = pd.DataFrame({
        "Component": ["Preventative Controls", "User Breach Cost", "Downtime Cost", "Total Incident Cost"],
        "Amount": [
            inputs['controls_cost'],
            user_breach_cost,
            downtime_cost,
            sle
        ]
    })

    # ROI Pie
    pie_data = pd.DataFrame({
        "Category": ["Preventative Controls", "Risk Reduction"],
        "Amount": [inputs['controls_cost'], risk_reduction]
    })

    # ALE bar
    ale_data = pd.DataFrame({
        "Scenario": ["Before Controls", "After Controls"],
        "ALE": [ale_before, ale_after]
    })

    # Baseline warning
    baseline_cost_per_day = inputs['revenue'] / 365
    if inputs['cost_per_day'] < baseline_cost_per_day:
        cost_warning = f"⚠️ Your estimated daily cost (${inputs['cost_per_day']:,.0f}) is below the baseline (${baseline_cost_per_day:,.0f})."
        cost_ok = False
    else:
        cost_warning = f"✅ Your estimated daily cost (${inputs['cost_per_day']:,.0f}) meets or exceeds the baseline (${baseline_cost_per_day:,.0f})."
        cost_ok = True

    return render_template(
        'index.html',
        inputs=inputs,
        sle=sle,
        ale_before=ale_before,
        ale_after=ale_after,
        risk_reduction=risk_reduction,
        roi=roi,
        ale_before_pct=ale_before_pct,
        ale_after_pct=ale_after_pct,
        risk_red_pct=risk_red_pct,
        cost_data=cost_data.to_dict(orient='records'),
        pie_data=pie_data.to_dict(orient='records'),
        ale_data=ale_data.to_dict(orient='records'),
        maturity_modifiers=maturity_modifiers,
        cost_warning=cost_warning,
        cost_ok=cost_ok
    )

if __name__ == '__main__':
    app.run(debug=True)
