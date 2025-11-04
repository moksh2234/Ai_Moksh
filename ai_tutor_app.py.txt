import streamlit as st
import sympy as sp
import wikipedia

st.set_page_config(page_title="AI Tutor (Offline)", layout="centered")
st.title("AI Tutor (Offline Version) 🤖📘")

st.markdown("""
This AI Tutor can solve math problems using **SymPy** and give science/general knowledge summaries using **Wikipedia**.
No API keys needed — works 100% offline (except Wikipedia lookups).
""")

# Input box
user_input = st.text_area("Enter your question:", height=120, placeholder="Example: Solve 2*x + 3 = 7 or What is photosynthesis?")

subject = st.selectbox("Select subject:", ["Auto-detect", "Math", "Science", "General Knowledge"])

def solve_math(expr_text):
    try:
        expr_text = expr_text.replace("^", "**")
        if "=" in expr_text:
            left, right = expr_text.split("=")
            left_expr = sp.sympify(left)
            right_expr = sp.sympify(right)
            eq = sp.Eq(left_expr, right_expr)
            symbols = list(eq.free_symbols)
            sol = sp.solve(eq, symbols[0]) if symbols else sp.solve(eq)
            return f"Equation solved:\n\n{symbols[0]} = {sol}"
        else:
            expr = sp.sympify(expr_text)
            simplified = sp.simplify(expr)
            numeric = None
            try:
                numeric = sp.N(expr)
            except Exception:
                pass
            out = f"Simplified expression:\n\n{simplified}"
            if numeric is not None:
                out += f"\n\nNumeric value: {numeric}"
            return out
    except Exception as e:
        return f"❌ Error: {e}"

def wiki_summary(query):
    try:
        return wikipedia.summary(query, sentences=3)
    except Exception:
        results = wikipedia.search(query)
        if results:
            try:
                return wikipedia.summary(results[0], sentences=3)
            except Exception:
                pass
        return "❌ No relevant Wikipedia summary found."

if st.button("Get Answer"):
    query = user_input.strip()
    if not query:
        st.warning("Please enter a question first.")
    else:
        is_math = subject == "Math" or (subject == "Auto-detect" and any(ch.isdigit() or ch in "+-*/^=()" for ch in query))
        if is_math:
            st.subheader("🧮 Math Solution")
            st.write(solve_math(query))
        else:
            st.subheader("📚 Wikipedia Answer")
            st.write(wiki_summary(query))

st.markdown("---")
st.markdown("Made by **Moksh’s AI Tutor** 💡")
