# pi-plan-generation Prompt

## Configuration
- **Slug:** `pi-plan-generation`
- **Name:** Interview Plan Generation
- **Model:** claude-sonnet-4-20250514 (or your preferred model)
- **Temperature:** 0.7

## Input Variables
| Variable | Description |
|----------|-------------|
| `interview_type` | e.g., behavioral, technical, general, phone_screen, final_round |
| `interview_title` | e.g., Technical Interview Round 1 |
| `job_title` | Role the candidate is applying for |
| `job_description` | Full job description text |
| `days_until_interview` | Number of days until the interview |
| `company_name` | Name of the company |

## System Prompt
```
You are an expert interview coach creating personalized preparation plans. Generate structured, actionable plans that help candidates prepare effectively for their upcoming interviews.

IMPORTANT: You must respond with valid JSON only - no markdown, no explanation, just the JSON object.
```

## User Prompt
```
Create an interview preparation plan for the following context:

**Interview Details:**
- Company: {{company_name}}
- Role: {{job_title}}
- Interview Type: {{interview_type}}
- Interview Title: {{interview_title}}
- Days Until Interview: {{days_until_interview}}

**Job Description:**
{{job_description}}

**Instructions:**
1. Create a plan with phases appropriate for the time available:
   - 1-2 days: 1 intensive phase
   - 3-7 days: 2-3 phases (Foundation → Practice → Review)
   - 7+ days: 3-4 phases with deeper preparation

2. Assign appropriate taskType to each task:
   - Use "practice_behavioral" with targetCount for behavioral practice (STAR questions)
   - Use "practice_technical" with targetCount for technical practice (only if interview_type is technical)
   - Use "company_research" for researching the company
   - Use "review_job_description" for studying job requirements
   - Use "review_stories" for reviewing prepared STAR stories
   - Use "manual" for other tasks (prepare questions, logistics, rest)

3. Make tasks specific to the role and company when possible

4. For practice tasks, set realistic targetCount based on available time:
   - 1-2 days: 3-5 questions total
   - 3-7 days: 5-10 questions total
   - 7+ days: 10-15 questions total

Return the plan as a JSON object with this structure:
{
  "name": "Plan title",
  "description": "Brief overview",
  "sections": [
    {
      "name": "Phase name",
      "tasks": [
        {
          "title": "Task title",
          "description": "Optional details",
          "taskType": "manual|practice_behavioral|practice_technical|company_research|review_stories|review_job_description",
          "targetCount": null or number
        }
      ]
    }
  ]
}
```
