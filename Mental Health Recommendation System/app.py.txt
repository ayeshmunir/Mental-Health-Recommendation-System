import pandas as pd
import joblib
import gradio as gr

model = joblib.load("mental_health_model.pkl")
label_encoders = joblib.load("label_encoders.pkl")

def combined_mental_health_academic_rule_system(user_input):
    recommendations = []
    stress = user_input.get("Stress_Level", 0)
    sleep = user_input.get("Sleep_Hours", 8)
    mood = user_input.get("Mood_Level", 5)
    exercise = user_input.get("Exercise", 1)

    if stress > 7 or mood < 4:
        mental_status = "High Risk"
        recommendations.append("Seek professional counseling or mental health support.")
        if sleep < 6:
            recommendations.append("Improve sleep routine and avoid late-night screen time.")
        if stress > 7:
            recommendations.append("Practice stress management techniques like meditation or breathing exercises.")
        if exercise < 1:
            recommendations.append("Engage in light physical activity to boost mood.")
    elif 4 <= stress <= 7 or 4 <= mood <= 6:
        mental_status = "Mild Risk"
        recommendations.append("Maintain a balanced lifestyle and monitor mental health.")
        recommendations.append("Engage in physical activity and social interaction.")
    else:
        mental_status = "Low Risk"
        recommendations.append("Your mental health appears stable.")
        recommendations.append("Continue healthy habits and self-care routines.")

    academic_responses = user_input.get("Academic_Responses", [0]*12)
    total_score = sum(academic_responses)

    if 0 <= total_score <= 7:
        academic_level = "Minimal Stress"
        recommendations.append("Student appears to be managing academic life well.")
        recommendations.append("Continue proper sleep, balanced study routines, and social interaction.")
    elif 8 <= total_score <= 14:
        academic_level = "Mild Stress"
        recommendations.append("Student may be experiencing some academic pressure.")
        recommendations.append("Improve time management and take short study breaks.")
    elif 15 <= total_score <= 25:
        academic_level = "Moderate Stress"
        recommendations.append("Student may be experiencing noticeable stress.")
        recommendations.append("Practice relaxation techniques such as deep breathing or mindfulness.")
        recommendations.append("Break tasks into smaller steps to reduce overwhelm.")
    elif 26 <= total_score <= 36:
        academic_level = "High Stress"
        recommendations.append("Student may be experiencing high academic stress.")
        recommendations.append("Seek support from mentors, teachers, counselors, or trusted individuals.")
    else:
        academic_level = "Invalid Score"
        recommendations.append("Please provide valid academic questionnaire responses (0-3 per question).")

    return {
        "Mental_Health_Status": mental_status,
        "Academic_Stress_Level": academic_level,
        "Recommendations": list(dict.fromkeys(recommendations))
    }

def gradio_interface(Stress_Level, Sleep_Hours, Mood_Level, Exercise,
                     Q1,Q2,Q3,Q4,Q5,Q6,Q7,Q8,Q9,Q10,Q11,Q12):

    user_input = {
        "Stress_Level": Stress_Level,
        "Sleep_Hours": Sleep_Hours,
        "Mood_Level": Mood_Level,
        "Exercise": Exercise,
        "Academic_Responses": [Q1,Q2,Q3,Q4,Q5,Q6,Q7,Q8,Q9,Q10,Q11,Q12]
    }

    result = combined_mental_health_academic_rule_system(user_input)

    if result["Mental_Health_Status"] == "High Risk":
        color = "#dc2626"
    elif result["Mental_Health_Status"] == "Mild Risk":
        color = "#f59e0b"
    else:
        color = "#16a34a"

    return f"""
    <div style="padding:20px;background:white;border-radius:10px;box-shadow:0 2px 8px rgba(0,0,0,0.1);">
    <h2>Mental Health Analysis Report</h2>
    <p><strong>Mental Health Status:</strong> <span style="color:{color};">{result["Mental_Health_Status"]}</span></p>
    <p><strong>Academic Stress Level:</strong> {result["Academic_Stress_Level"]}</p>
    <h3>Recommendations</h3>
    <ul>
    {''.join(f"<li>{r}</li>" for r in result["Recommendations"])}
    </ul>
    </div>
    """

questions = [
    " I feel stressed or pressured because of my academic workload.",
    " I find it difficult to concentrate during lectures or while studying.",
    " I feel anxious or worried about exams or assignments.",
    " I feel mentally exhausted even after resting.",
    " I have trouble sleeping or do not get enough sleep at night.",
    " I feel overwhelmed when managing multiple academic tasks.",
    " I feel discouraged or lose motivation to study.",
    " I feel nervous when thinking about my academic performance or future career.",
    " I feel isolated or disconnected from classmates or friends.",
    " I find it difficult to maintain a balance between studies and personal life.",
    " I feel that my daily routine is disorganized or difficult to manage.",
    " I often feel negative or low in mood during the day."
]

inputs = [
    gr.Slider(0,10,label="Stress Level"),
    gr.Slider(0,12,label="Sleep Hours"),
    gr.Slider(0,10,label="Mood Level"),
    gr.Slider(0,3,label="Exercise Level")
]

for q in questions:
    inputs.append(gr.Slider(0,3,label=q))

iface = gr.Interface(
    fn=gradio_interface,
    inputs=inputs,
    outputs=gr.HTML(),
    title="Mental Health Recommendation System"
)

iface.launch((
    server_name="0.0.0.0",
    server_port=7860
)