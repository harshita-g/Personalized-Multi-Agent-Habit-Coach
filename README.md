# Personalized-Multi-Agent-Habit-Coach
<br> Agents Intensive - Capstone Project </br>
<br> Hackathon Writeup · Nov 30, 2025 </br>
<br>A multi agent system that learns your habits, evaluates your progress, and generates actionable behavior plans. </br>

<br>**Problem Statement**</br>
Building and sticking to good habits sounds simple, but in real life it is messy. People try to exercise, study, sleep better, or meditate, but they often lose track of what they actually did, whether they are improving, and how far they are from their goals. Most habit apps only collect data (“I checked the box”), but they do not really help users think about their behavior or turn those logs into realistic plans.

The problem I am tackling is:
How can we give an individual a personal “habit coach” that understands natural language, remembers their history, and turns their own data into actionable, realistic plans?

I think this is interesting because it sits at the intersection of:

Self-improvement (habits, goals, routines)

Reasoning (identifying patterns and bottlenecks)

Planning (generating concrete next steps instead of vague motivation)

Instead of just tracking streaks, I want an agent that can answer questions like:

“How consistent was I with workouts this week?”

“How does my energy change with different habits?”

“What’s a realistic plan for next week based on what I actually did, not what I wish I did?”

<br>**Why agents?**</br>
A static app could log habits and show charts, but it can not:

flexibly understand messy natural language (“I dragged myself through a 20-minute run and felt dead”),

decide which actions to take (log this, analyze that, or generate a plan),

or adapt the conversation based on the user’s goals and history.

Agents are a good fit here because they can combine:

Natural language understanding
The user can speak like a human, not like a form:
“I studied Science for 45 minutes but felt tired.”
An agent can parse that into structured fields (habit, duration, energy, rating, notes).

Tool use over structured memory
The tools I built (for logging, reading logs, computing progress vs goals, summarizing energy) let the agents ground their reasoning in actual past data, not hallucinations.

Specialized roles
Different parts of the problem need different “mindsets”:

-- Logging behavior
-- Doing analysis
-- Giving coaching and planning advice
It maps naturally to multiple specialized agents instead of one giant prompt.

Interactive planning
The user can go back and forth:
“That plan is too heavy on weekdays, can we shift more to the weekend?”
The Planner agent can adjust while staying grounded in the same underlying data.
So agents are not just a fancy wrapper here, they are what allow the system to listen, remember, analyze, and coach in a way that feels personal and adaptive rather than like a static dashboard.
What you created
I created a multi agent habit coaching system with three main LLM agents built using Google ADK:

LoggingAgent – “The Listener”
Takes natural language inputs like “I worked out 30 minutes today and felt great.”

Extracts structured fields: date, habit, category, duration_minutes, rating, energy_level, notes.
Calls tools to store those entries into a JSON “habit database”.
Also handles goal setting, e.g., “I want to study 10 hours a week.”

InsightAgent – “The Analyst”
Reads the last N days of logs and the user’s goals.
Calls analysis tools to compute total sessions and minutes per habit, progress vs goals, simple consistency scores, average ratings and energy.
Combines everything into a single JSON summary that captures the user’s current state.

PlannerCoachAgent – “The Coach”
Takes the JSON summary from InsightAgent as input.
Generates a personalized, realistic plan (e.g., next 7 days) that:
doesn’t assume the user will suddenly 5x their effort,
aligns with when they tend to have more energy,
focuses on a few key habits, and
proposes fallback versions for low-energy days.

Under the hood:
-- Memory is stored in JSON files (habit_database.json and habit_goals.json), which act as long-term habit/goal storage.
-- Tools (wrapped with FunctionTool) are the bridge between the LLM agents and this structured memory (e.g., add_habit_entry, set_goal, analyze_progress_against_goals, summarize_energy_patterns).

-- InMemoryRunner manages sessions so each agent can have a coherent multi-turn conversation with the user.

The overall flow looks like:
User → LoggingAgent → JSON DB → InsightAgent → JSON summary → PlannerCoachAgent → Personalized plan

<br>**Demo**</br>
Here is how a typical end-to-end interaction with the system looks:

1. User logs habits via LoggingAgent

2. User asks for insights via InsightAgent

The JSON includes fields like- actual vs target sessions/minutes per habit, consistency scores, average energy per habit and “high_level_findings”

3. User asks for a plan via PlannerCoachAgent

In the codebase, I provide a demo_multi_agent_flow() function that runs this full pipeline:
Logging some example habits and goals
Calling InsightAgent to produce a summary
Passing that summary into PlannerCoachAgent to generate a concrete 7-day plan
I also have a small evaluation set that runs test prompts through each agent (logging, insight, planning) to verify that they behave as expected.

<br>**The Build**</br>
This project is built on top of Google’s Agent Development Kit (ADK) and Gemini, inside a Kaggle notebook.

Key pieces:
Language Model:
gemini-2.5-flash-lite via google-genai
Wrapped with LlmAgent from google.adk.agents
Agent Framework (ADK):

LlmAgent: defines each agent’s instructions, tools, and model
FunctionTool: wraps Python functions so the LLM can call them as tools
InMemoryRunner: handles sessions and streaming responses
ToolContext: passes metadata into tools if needed
Tools implemented in Python:

add_habit_entry: append structured entries to habit_database.json
-set_goal / get_goals: store and retrieve habit goals from habit_goals.json
get_entries_last_n_days / get_entries_by_date: return human-readable summaries
analyze_progress_against_goals: aggregate sessions/minutes, compute simple consistency vs goals
summarize_energy_patterns: compute average ratings and energy per habit
Memory / Storage:

JSON files in the Kaggle working directory act as persistent memory for habits and goals.
The structure is keyed by USER_ID and date, with lists of habit entries.
Orchestration and Demo:

Helper function run_agent_dialog to talk to each agent independently.
demo_multi_agent_flow() to run the full pipeline: logging → insights → planning.
A small evaluation harness (run_trimmed_evaluation) to test key scenarios.
Everything is written in Python in a single Kaggle notebook, with comments explaining the main parts of the architecture and how the agents interact.
