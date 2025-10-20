# Creating an AI Assisted development work flow

I am a senior software developer with backend, frontend and design experience. I want to make use of AI for software development. I am trying out cursor to see if it will create good results. Note this is not about vibe coding in the 'traditional' sense where you create an application without even looking at the code. I want to have some level of control over the architecture design, separation of concerns, libraries being used and best practices for the language, framework and libraries. 

I have thought of a development process that I think could work. The details still need to be filled in 

## Development steps

Fist of all, the backend and frontend will be separate projects that can be developed with the help of AI independently, with a clear defined API between them.

### The backend

1. Create a starter project for the desired framework, for instance Python FastAPI or Nest.js
2. Manually define an architecture and class structure common for the used framework.
3. Compose a requirements document that describes the requirements:
 a. The main purpose of the application.
 b. The different layers of the application, for instance controllers, services, repositories.
 c. Requirements for each endpoint with input and output models, validation, business logic, data storage, etc.
4. Give the AI Agent the task to implement the endpoints.

### The frontend

1. Create Figma designs for the web or mobile app.
2. Create a starter project for the desired framework, for instance Next.js or React Native Expo.
3. Compose a requirements document that describes the requirements:
 a. The main purpose of the application.
 b. The different screens of the applications and how they interact with each other.
 d. Where data is fetched from the backend and where data is sent to the backend.
4. Use the Figma MCP connection with Cursor to let the AI Agent implement React (Native) pages or screens from the Figma designs.
5. Give the AI Agent the task to implement all logic and backend requests.

## Instruction aspects

To increate the likelihood of getting to the desired result there are 3 aspects of the instructions for the AI Agent. These are the rules, the scope of the tasks and the level of detail in the requirements. I think these aspects have the biggest impact on the outcome.

### Rules

Cursor lets you create rule files that give the AI Agent context and direction in the way the task should be performed. This could be (among others) conceptually, like a thinking process or steps to perform the task, or more specific like design patterns, coding style or the preference for certain libraries.

### Scope of the tasks

You can give the AI Agent one large task which should result in a mostly complete application, adhering to all requirements, or you can split the task up into smaller tasks, for instance a task per endpoint or screen and other tasks for things like event tracking, logging, error handling etc.

### Level of detail in the requirements

You can make the requirements very detailed and specific to make sure the AI Agent implements the application exactly as you want, or you can make the requirements a bit more high level, to leave room for interpretation.

 
## My Goal

I want to create an optimal work flow by clearly defining the development steps for the backend and frontend, and by filling in the 3 instruction aspects in the best way possible. 

## My thoughts

I have some thoughts on the development steps and the instruction aspects. Please take my thoughts into consideration but do not rely too heavily on them. They are mostly based on intuition and years of development experience. I have developed a preferred way of working but I am flexible and open to improvement.

### Development steps

#### Backend development steps

Of all things, for this I feel the most certain that I am on the right path. Creating a framework/architecture the AI Agent can work in will make sure that the solution reflects the way I would logically build the backend application myself. Often, requirements for backend applications can be defined quite exact so having a detailed level of requirements would fit here. One question here would be related to instruction aspect 2, Scope of the tasks. Should the backend be implemented in one task or split up into multiple smaller tasks?

#### Frontend development steps

The part of creating components based on Figma designs is to me the most experimental. Is it possible to get decent results and how should the task be formulated to get good results? Should I just give a link to the Figma frame or should this be accompanied by a textual representation of the screen and its (reusable) components, user interactions ect. Related to instruction aspect 2, should the task be split up in smaller tasks, for instance a task per screen?

### Instruction aspects

#### Rules

This part is the most vague for me. What should be in the rules? I have seen people formulate complete operational doctrines with different phases like analysis, planning, operation and verification. Is this helpful or overkill? Normally I am a fan of concise and to the point, but also sound and complete instructions. I guess stating clear steps and goals could work well. Also, because the rules are used as input for the LLM, they count for the number of input tokens, which influences the cost of using the LLM. Keeping the rules shorter will be beneficial in that regard. With respect to design patterns, should important design patterns be referenced or should I just state the fact that important design patterns need to be adhered to?

#### Scope of the tasks

I am leaning towards the idea of splitting up the task into smaller tasks. This way I can describe the task in more detail and the AI Agent can have more focus. These smaller tasks should each result in a testable deliverable so I can verify their correctness.

#### Level of detail in requirements

Here I am leaning towards the idea that more detail is better. This will increase the chances that the application is implemented exactly the way I want it to be. It could be that more flexibility helps with the frontend application. Firstly because describing all UI state changes in detail is a lot of work and secondly because UX design is neither my specialty nor an exact science, so leaving room for the AI Agent to improvise here could be beneficial.

### The task

I need you to analyze the development steps and the instruction aspects, including my thoughts on them. Then provide an optimal development work flow with development steps. Provide a recommended approach for each of the instruction aspects. Describe key considerations and pros, cons and pitfalls for the given approach. Furthermore, let me know if there are any other important aspects of the instructions that I missed.

===

TODO:

- Additionally ask Reddit to 'edit' the story for an optimal LLM answer
- Create a git repository where the story, AI answer and resulting way of working are defined.

