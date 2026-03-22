A multi-agent system designed to automate the creation of academic courses. Given inputs from a professor, it generates a complete course package including class plans, lecture notes, slides, exercises, exams, and homework projects.

---

## Course Agents

1. **Planner / Orchestrator**
2. **Content Collector**
3. **Class Builders** — see [Class Agents](#class-agents)
4. **Exam Builder**
5. **Homework Project Builder**
6. **Translator**

---

### 1. Planner / Orchestrator

Reads all input from the professor and builds the course plan. Validates consistency across classes, exams, and homework projects. Ensures the course structure matches the program requirements.

Ask questions to the professor:
- Program (default: Computer Science)
- Course name
- Syllabus
- Course curriculum (if available)
- Class time length (default: 3h30)
- Number of classes (default: 16)
- Number of exams (default: 2 — one midterm, one final)
- Number of homework projects (default: 3)
- Any additional information relevant to the course
- Language for content delivery

Output: save all inputs as a shared context document used by all downstream agents.

---

### 2. Content Collector

Asks the professor whether existing class materials are available and ingests them as reference content for all downstream agents.

Ask questions to the professor:
- Do you have an existing class plan (syllabus breakdown per class)?
- Do you have content prepared for any of the classes? (slides, notes, scripts, etc.)
- For each class with existing content, what format is it in?

Supported input formats:
- PDF documents
- Office files (`.pptx`, `.docx`, `.xlsx`)
- Markdown files (`.md`)
- Web pages (URLs)
- Apple formats (`.pages`, `.key`, `.numbers`)

For each piece of content provided:
- Extract and store the text, structure, and key concepts
- Tag it with the class number or topic it belongs to
- Save it to a shared content repository accessible by all other agents

Output: a structured content index per class, linking raw source files and extracted summaries. Downstream agents must consult this index before generating new content — existing material takes priority over generating from scratch.

---

### 3. Class Builders

Described in detail in [Class Agents](#class-agents).

---

### 4. Exam Builder

Uses the content taught up to the exam date to build a closed-book assessment.

Rules:
- Questions must be medium or hard difficulty
- The exam must fill the entire class time slot
- Students cannot consult notes
- The exam must include supporting information (formulas, tips, reference tables) so students can solve problems without memorization

Ask questions to the professor:
- Does the professor have an exam template?
- How many questions are expected?
- What structure should the exam follow? What should each question assess?
- How many different exam versions are needed, and what sections should differ between versions?

Output: one or more exam documents in markdown format, ready for PDF export.

---

### 5. Homework Project Builder

Uses the content taught up to that point to create a real-world practical challenge.

Rules:
- Create a central challenge with hard difficulty
- Build 15 real-world case variations on top of the central challenge
- Write clear expectations for what students must deliver
- Define evaluation criteria
- Specify deliverables and submission format

Ask questions to the professor:
- Is this a programming project?
  - If yes: what is the expected programming language?
  - Can students use external libraries? Which ones are allowed?

Output: a homework document in markdown format with the challenge description, cases, rubric, and submission instructions.

---

### 6. Translator

Translates all course artifacts into the language defined in the Planner context. Ensures terminology and wording are consistent across all documents.

Trigger: runs after all other agents have produced their output, or on demand per artifact.

---

## Class Agents

### 1. Class Builder (Orchestrator)

Coordinates all class-level agents. Validates that all content is consistent and fits within the expected class time. Signals failure if time budget is exceeded or content is inconsistent.

Uses the agents below to build a complete class.

---

### 2. Scriptwriter / Professor Guide

Creates a step-by-step class guide for the professor, including total class time and time allocated to each activity (theory, exercises, breaks, Q&A). Built from lecture notes and exercises.

---

### 3. Lecture Notes Builder

Explains the theory in detail with examples. Includes resolved theoretical exercises (start with easy, then medium level). Adjusts depth to the audience level. Fetches original papers and references.

Requirements:
- Format: markdown
- Use LaTeX for formulas and equations
- Use Mermaid for diagrams
- Include exercises from the Theoretical Exercises builder
- Output: a standalone document per class

---

### 4. Code Example Builder

Builds a working code example illustrating a practical application of the class content. Only runs if the class has a practical aspect.

Requirements:
- Creates a dedicated folder for the code
- Documents what is needed to run the code (local setup or external resources)
- Based on the lecture notes
- Must use a real-world scenario
- Up to 3 code examples per class

---

### 5. Slide Builder

Builds a slide deck from the theory and resolved exercises.

Requirements:
- Format: markdown
- Use LaTeX for formulas and equations
- Use Mermaid for diagrams
- Based on the lecture notes
- Include exercises from the lecture notes
- Output: one slide deck per class

---

### 6. Theoretical Exercises Builder

Creates pen-and-paper exercises for students to solve in their notebooks, suitable for whiteboard resolution by the professor.

Requirements:
- At least 15 exercises
- Mix of easy, medium, and hard difficulty levels

---

### 7. Practical Exercises Builder

Creates a hands-on coding exercise where students apply what they learned.

Requirements:
- Simple real-world scenario
- Students must deliver a code artifact
- Supported environments: Google Colab (Jupyter), Python, or the language defined in the course context
- List all required libraries and resources
- Provide a boilerplate for students to continue from
- Reuse code from the Code Example Builder where possible

---

### 8. Extra Material

Curates supplementary resources for students who want to go deeper.

Requirements:
- Include YouTube videos, articles, and web pages
- Select only high-quality, relevant content
- Output: a curated list with links and one-line descriptions per resource

---

## Skills

> These are initial skill suggestions. More skills may be identified as the system evolves.

### Skill: Code Builder *(suggested)*
- Use the programming language defined in the course context
- Write detailed inline comments to support student understanding
- Summarize relevant theory in key code blocks

### Skill: Slide Builder *(suggested)*
- Write all content in markdown format
- Follow the template provided by the professor
- Export to PDF at the end

### Skill: Diagrams *(suggested)*
- Preferred tool: Mermaid
- Fallback: ASCII art
- Alternative: generate using GenAI

### Skill: Graphs *(suggested)*
- Preferred tool: Mermaid
- Alternative: generate using GenAI

### Skill: Image Builder *(suggested)*
- Generate using GenAI

### Skill: Exercise Generator *(suggested)*
- Generate exercises from a given topic, concept, or learning objective
- Support multiple formats: multiple choice, open-ended, fill-in-the-blank, true/false
- Tag each exercise with difficulty level (easy / medium / hard) and the concept it covers
- Ensure variety — avoid repeating the same structure back-to-back

### Skill: Rubric Builder *(suggested)*
- Create grading rubrics for exercises, homework, and exams
- Define clear criteria and point weights per criterion
- Align rubric with the stated learning objectives

### Skill: Learning Objective Writer *(suggested)*
- Write clear, measurable learning objectives for a class or unit
- Follow Bloom's taxonomy levels (remember, understand, apply, analyze, evaluate, create)
- Used by other agents to calibrate content depth and assessment difficulty

### Skill: Citation & Reference Finder *(suggested)*
- Search for and format academic references related to a given topic
- Prefer primary sources (papers, textbooks) over secondary ones
- Output in a consistent citation format (APA, ABNT, or professor's preference)

### Skill: Time Estimator *(suggested)*
- Estimate the time required to cover a given piece of content in class
- Takes into account content complexity, audience level, and activity type (lecture, exercise, discussion)
- Used by the Scriptwriter and Class Builder to validate time budgets

### Skill: Summary Writer *(suggested)*
- Produce concise summaries of lecture notes or topics
- Useful for slide introductions, extra material descriptions, and review sections
- Adjust length based on context (one paragraph, bullet list, or executive summary)

### Skill: Boilerplate Generator *(suggested)*
- Generate starter code scaffolds for practical exercises and homework
- Include placeholder comments indicating what the student must implement
- Match the style and language of the Code Example Builder output

### Skill: PDF Reader *(suggested)*
- Extract text, structure, and embedded images from PDF files
- Preserve section hierarchy (headings, paragraphs, lists) where possible
- Handle scanned PDFs using OCR when native text is unavailable
- Output: structured text ready for ingestion by other agents

### Skill: Office Format Reader *(suggested)*
- Extract content from Office and Apple formats: `.pptx`, `.docx`, `.xlsx`, `.pages`, `.key`, `.numbers`
- For presentations (`.pptx`, `.key`): extract slide titles, body text, speaker notes, and diagrams
- For documents (`.docx`, `.pages`): extract text with heading structure preserved
- For spreadsheets (`.xlsx`, `.numbers`): extract tables and labels
- Output: structured text and tables ready for ingestion by other agents

### Skill: Web Page Reader *(suggested)*
- Fetch and extract the main content from a given URL
- Strip navigation, ads, and boilerplate; preserve article structure
- Follow links one level deep if the professor provides an index page
- Output: clean markdown text ready for ingestion by other agents

---

## Next Steps

- Define agent descriptions and skill descriptions in detail
- Identify and document the tools to be used for content production
