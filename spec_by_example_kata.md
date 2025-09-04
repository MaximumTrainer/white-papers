Here is a comprehensive Specification by Example practice kata designed to foster collaboration, refinement, and the creation of clear acceptance criteria through the 3 Amigos process.

### **Kata Title: The Loan Application Pre-Qualifier**

**Objective:** To practice collaborative refinement, uncover hidden assumptions, and transform a vague requirement into a set of clear, testable acceptance criteria using concrete examples.

**Participants:** The "3 Amigos" – ideally a Developer, a Tester, and a Product Owner (or Business Analyst). Can be run with 2-4 people.

**Time Box:** 45-60 minutes

---

### 1. The Backlog Item (The "What")

**User Story:** As a loan applicant, I want to get an immediate pre-qualification result online so that I can understand my chances before submitting a full application.

**Initial, Vague Acceptance Criteria (Provided by the PO):**
*   AC1: The system must check the applicant's credit score.
*   AC2: The system must consider the applicant's debt-to-income ratio.
*   AC3: The system must return a "Prequalified" or "Not Prequalified" result.

---

### 2. The Exercise (The "How")

**Part 1: Individual Preparation (5 minutes)**
*   Each participant reads the backlog item and initial ACs silently.
*   Each person writes down at least 2-3 questions they have and any assumptions they are making. (e.g., "What is a 'good' credit score?", "How is debt-to-income ratio calculated?").

**Part 2: 3 Amigos Discussion & Question Storm (20 minutes)**
*   The team meets. The Product Owner starts by briefly explaining the business goal.
*   **Facilitator's Goal:** Uncover all hidden rules and assumptions. *Do not write solutions or final ACs yet. Just ask questions.*
*   **Sample Questions to Prompt Discussion:**
    *   What are the specific thresholds for credit score? (e.g., Is it a tiered system? 600-679 Fair, 680-739 Good, 740+ Excellent?)
    *   How is Debt-to-Income (DTI) Ratio calculated? (Formula: Total Monthly Debt Payments / Gross Monthly Income)
    *   What is the maximum allowed DTI?
    *   Are there any other factors? (e.g., minimum income? age? loan amount?)
    *   What if the credit score is excellent but the DTI is terrible, or vice versa?
    *   Is this a simple "AND" check, or a more complex rules engine?
    *   What are the sources of data? (Assume we can get accurate credit score and DTI for this exercise).

**Part 3: Specification by Example - Creating a Decision Table (20 minutes)**
*   Now, collaborate to define the specific rules using examples. The tester often leads this part.
*   **The Rule:** After discussion, the team agrees on the following business rule:
    *   To be prequalified: Applicant must have **AT LEAST a "Good" credit score (≥ 680) AND a DTI ratio ≤ 36%.**

*   **Team Task:** Create a table of concrete examples that describe the system's behavior. This table *becomes* your acceptance criteria.

| Example # | Credit Score | DTI Ratio | Expected Result | Notes / Rule Explained |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 720 | 30% | Prequalified | Both conditions are met (Good score, DTI ≤ 36%). |
| 2 | 720 | 40% | Not Prequalified | Good score, but DTI is too high. |
| 3 | 650 | 30% | Not Prequalified | DTI is good, but credit score is "Fair" (< 680). |
| 4 | 650 | 40% | Not Prequalified | Neither condition is met. |
| 5 | 680 | 36% | Prequalified | Boundary testing: Score is at minimum "Good", DTI is at maximum. |
| 6 | 679 | 36% | Not Prequalified | Boundary testing: Score is just below "Good". |
| 7 | 680 | 36.1% | Not Prequalified | Boundary testing: DTI is just above 36%. |

**Part 4: Formulating Final Acceptance Criteria (10 minutes)**
*   Based on the examples and rules you've discovered, rewrite the original vague ACs into clear, testable statements.

**Final, Testable Acceptance Criteria:**
*   **GIVEN** I am a loan applicant
    *   **AC1 (Happy Path):** WHEN my credit score is 680 or higher **AND** my Debt-to-Income ratio is 36% or less, THEN the system **MUST** return a "Prequalified" result.
    *   **AC2 (Alternative Path 1):** WHEN my credit score is 680 or higher **BUT** my Debt-to-Income ratio is greater than 36%, THEN the system **MUST** return a "Not Prequalified" result.
    *   **AC3 (Alternative Path 2):** WHEN my credit score is less than 680 **BUT** my Debt-to-Income ratio is 36% or less, THEN the system **MUST** return a "Not Prequalified" result.
    *   **AC4 (Boundary Testing):** The system must correctly handle boundary values (e.g., credit score of 680 and DTI of 36.0% qualify; credit score of 679 and DTI of 36.1% do not qualify).

---

### 3. debrief & Learning Outcomes (5-10 mins)

Discuss as a team:
1.  **How did the initial ACs compare to the final ones?**
2.  **Whose perspective (Dev, Test, PO) was most valuable in uncovering which types of questions?**
3.  **Was there a "aha!" moment where an assumption was wrong?**
4.  **How does using a table of examples change the way you think about the requirement?**
5.  **How will this improve the development and testing process later?**

### **Facilitator's Notes & How to Scale:**

*   **For Beginners:** Provide them with the exact business rule ("Good score AND DTI ≤ 36%") to help them start creating examples.
*   **For Advanced Practitioners:** Make the problem more complex. Add a third rule (e.g., "OR if the applicant has a DTI ≤ 28%, a 'Fair' credit score (600-679) is acceptable"). This introduces OR logic and makes the example table even more critical.
*   **Remote Teams:** Use a collaborative tool like Miro, Jamboard, or Confluence for building the table together in real-time.
*   **The Key Takeaway:** The goal isn't to perfectly design a loan system. The goal is to practice the conversation that prevents misunderstandings and creates executable specifications. The conversation *is* the refinement.
