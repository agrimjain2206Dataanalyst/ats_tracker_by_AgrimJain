
from dotenv import load_dotenv
import streamlit as st
import google.generativeai as genai
import pdf2image
from PIL import Image
import os
import io
import base64

# Load API key
load_dotenv()
genai.configure(api_key=os.getenv("Google_API_Key"))

# Convert PDF to image
def input_pdf_setup(uploaded_file):
    #poppler bin path
    poppler_path = r"C:\Users\adity\Downloads\Release-24.08.0-0\poppler-24.08.0\Library\bin"

    images = pdf2image.convert_from_bytes(
        uploaded_file.read(),
        poppler_path=poppler_path
    )
    first_page = images[0]
    
    img_byte_arr = io.BytesIO()
    first_page.save(img_byte_arr, format="JPEG")
    img_byte_arr = img_byte_arr.getvalue()

    pdf_parts = [
        {
            "mime_type": "image/jpeg",
            "data": base64.b64encode(img_byte_arr).decode()
        }
    ]
    return pdf_parts, first_page

# Gemini response
def get_gemini_response(job_desc, pdf_parts, prompt):
    model = genai.GenerativeModel('gemini-1.5-flash')  # reliable model
    response = model.generate_content([prompt, job_desc, pdf_parts[0]])
    return response.text

# Streamlit UI
st.set_page_config(page_title="ATS Resume Expert")
st.title("📄 ATS Resume Expert")

job_desc = st.text_area("Job Description", key="input", height=150)
uploaded_file = st.file_uploader("Upload your resume (PDF)", type=["pdf"])

if uploaded_file:
    st.success("PDF uploaded successfully ✅")

submit1 = st.button("Tell Me About The Resume")
submit2 = st.button("Percentage Match")

prompt1 = """
You are an experienced HR Manager with expertise in evaluating resumes across multiple industries and job domains.  
Review the candidate’s resume against the given job description.  

Provide a clear evaluation that includes:  
1. Key strengths and relevant skills of the candidate.  
2. Areas of improvement or missing elements compared to the job description.  
3. An overall suitability score (0–100) based on alignment with the role.  
4. Practical and actionable suggestions to improve the resume for better chances of selection.  
Keep the feedback professional, easy to understand, and applicable to both technical and non-technical roles.
"""

prompt2 = """
You are an ATS (Applicant Tracking System).
Your task is to evaluate a candidate’s resume against the provided job description.
Please provide the following:
Percentage Match – How closely the resume aligns with the job description (0–100%).
Missing Keywords/Skills – Important role-specific terms, skills, or qualifications that are present in the job description but missing in the resume.
Final Thoughts – A brief evaluation of the candidate’s suitability for the role, highlighting whether they are a strong, average, or weak fit.
"""

if submit1 and uploaded_file:
    pdf_parts, preview = input_pdf_setup(uploaded_file)
    response = get_gemini_response(job_desc, pdf_parts, prompt1)
    st.image(preview, caption="Resume Preview", use_column_width=True)
    st.subheader("Response")
    st.write(response)

elif submit2 and uploaded_file:
    pdf_parts, preview = input_pdf_setup(uploaded_file)
    response = get_gemini_response(job_desc, pdf_parts, prompt2)
    st.image(preview, caption="Resume Preview", use_column_width=True)
    st.subheader("Response")
    st.write(response)

# Footer
st.markdown("""
    <div class="footer">
        Powered by <span>Agrim Jain</span> © All rights reserved
    </div>
""", unsafe_allow_html=True)
