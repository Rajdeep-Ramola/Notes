# Building Agents with LiveKit Framework

## Features of the Framework

- **Voice, video, and text**: Build agents that can process realtime input and produce output in any modality.
- **Tool use**: Define tools that are compatible with any LLM, and even forward tool calls to your frontend.
- **Multi-agent handoff**: Break down complex workflows into simpler tasks.
- **Extensive integrations**: Integrate with nearly every AI provider there is for LLMs, STT, TTS, and more.
- **State-of-the-art turn detection**: Use the custom turn detection model for lifelike conversation flow.
- **Made for developers**: Build your agents in code, not configuration.
- **Production ready**: Includes built-in agent server orchestration, load balancing, and Kubernetes compatibility.

## How an Agent Starts Up

1. When our agent code starts, it first registers with a LiveKit server to run as an agent server process.
2. Our agent server waits until it receives a dispatch request.
3. To fulfill this request, the agent server boots a job subprocess which joins the room.
4. After the agent joins the room, the agent and our frontend app can communicate using LiveKit WebRTC.

## Example: STT-LLM-TTS Pipeline

```python
from dotenv import load_dotenv
from livekit import agents
from livekit.agents import (
    AgentServer,
    AgentSession,
    Agent,
    inference,
    room_io,
    TurnHandlingOptions,
)
from livekit.plugins import ai_coustics

# Load API keys and other environment variables
load_dotenv(".env.local")

class Assistant(Agent):
    def __init__(self) -> None:
        super().__init__(
            # System prompt that defines the agent's personality and behavior
            instructions="""
            You are a helpful voice AI assistant.
            You eagerly assist users with their questions by providing information from your extensive knowledge.
            Your responses are concise, to the point, and without any complex formatting or punctuation including emojis, asterisks, or other symbols.
            You are curious, friendly, and have a sense of humor.
            """,
        )

# Create the LiveKit agent server
server = AgentServer()

@server.rtc_session(agent_name="my-agent")
async def my_agent(ctx: agents.JobContext):
    """
    This function runs whenever a new RTC session is created.

    Architecture:
        User Speech
            ↓
        STT (Deepgram)
            ↓
        LLM (OpenAI)
            ↓
        TTS (Cartesia)
            ↓
        Agent Speech
    """
    session = AgentSession(
        # Convert user speech into text
        stt=inference.STT(
            model="deepgram/nova-3",
            language="multi",
        ),
        # Generate textual responses
        llm=inference.LLM(
            model="openai/chat-latest",
        ),
        # Convert generated text back to speech
        tts=inference.TTS(
            model="cartesia/sonic-3",
            voice="9626c31c-bec5-4cca-baa8-f8ba9e84c8bc",
        ),
        # Detect when the user has finished speaking
        # before sending the transcript to the LLM
        turn_handling=TurnHandlingOptions(
            turn_detection=inference.TurnDetector(),
        ),
    )

    # Start the voice session and connect the agent to the room
    await session.start(
        room=ctx.room,
        agent=Assistant(),
        room_options=room_io.RoomOptions(
            audio_input=room_io.AudioInputOptions(
                # Enhance incoming microphone audio
                # to reduce background noise
                noise_cancellation=ai_coustics.audio_enhancement(
                    model=ai_coustics.EnhancerModel.QUAIL_VF_S
                ),
            ),
        ),
    )

    # Send an initial instruction so the agent speaks first
    await session.generate_reply(
        instructions="Greet the user and offer your assistance."
    )

if __name__ == "__main__":
    # Start the LiveKit agent server
    agents.cli.run_app(server)
```

## Example: Realtime Speech-to-Speech Model

```python
from dotenv import load_dotenv
from livekit import agents
from livekit.agents import AgentServer, AgentSession, Agent, room_io
from livekit.plugins import (
    openai,
    ai_coustics,
)

# Load API keys and other environment variables
load_dotenv(".env.local")

class Assistant(Agent):
    def __init__(self) -> None:
        super().__init__(
            # Basic system instructions for the assistant
            instructions="You are a helpful voice AI assistant."
        )

# Create the LiveKit agent server
server = AgentServer()

@server.rtc_session(agent_name="my-agent")
async def my_agent(ctx: agents.JobContext):
    """
    This function runs whenever a client joins the RTC room.

    Architecture:
        User Speech
            ↓
        OpenAI Realtime Model
            ↓
        Agent Speech

    Unlike the previous example:
    - No separate STT model
    - No separate LLM model
    - No separate TTS model

    The realtime model handles speech understanding,
    reasoning, and speech generation internally.
    """
    session = AgentSession(
        llm=openai.realtime.RealtimeModel(
            # Voice used for synthesized responses
            voice="coral",
        )
    )

    # Connect the realtime model to the room
    await session.start(
        room=ctx.room,
        agent=Assistant(),
        room_options=room_io.RoomOptions(
            audio_input=room_io.AudioInputOptions(
                # Clean up incoming audio before it reaches
                # the realtime model
                noise_cancellation=ai_coustics.audio_enhancement(
                    model=ai_coustics.EnhancerModel.QUAIL_VF_S
                ),
            ),
        ),
    )

    # Trigger an initial spoken greeting
    await session.generate_reply(
        instructions="""
        Greet the user and offer your assistance.
        You should start by speaking in English.
        """
    )

if __name__ == "__main__":
    # Start the LiveKit agent server
    agents.cli.run_app(server)
```

## Logic and Structure of Building Agents

| Component | Description | Use cases |
| --- | --- | --- |
| **Agent sessions** | Orchestrate input collection, pipeline management, and output delivery. The main orchestrator for your voice AI app. | Single-agent apps, session lifecycle management, and room I/O configuration. |
| **Chat context** | Manage the conversation history sent to the LLM on each turn. Create, copy, truncate, and merge contexts to control what the model knows. | Initializing context with user data, preserving history across handoffs, and injecting per-turn context. |
| **Tasks & task groups** | Create focused, reusable units that perform specific objectives and return typed results. Tasks run inside agents and take temporary control until completion. | Consent collection, structured data capture, and multi-step processes with task groups. |
| **Workflows** | Model repeatable patterns with agents, handoffs, and tasks for complex voice AI systems. | Multi-persona systems, conversation phase management, and specialized agent routing. |
| **Tool definition & use** | Extend agent capabilities with custom functions callable by the LLM for external actions and data access. | API integrations, frontend RPC calls, and triggering agent handoffs. |
| **Pipeline nodes & hooks** | Customize agent behavior at pipeline processing points with custom STT, LLM, TTS, and lifecycle hooks. Override nodes to modify input, output, or add custom logic. | Custom providers, output modification, and pronunciation control. |
| **Turn detection & interruptions** | Manage conversation flow with turn detection, interruption handling, and manual turn control. | Natural conversation timing, interruption management, and push-to-talk interfaces. |
| **Agents & handoffs** | Define distinct reasoning behaviors and transfer control between agents when different capabilities are needed. | Role-based agents, model specialization, and permission management. |
| **External data & RAG** | Connect agents to external data sources, databases, and APIs for RAG and data operations. Load initial context, perform RAG lookups, and integrate with external services. | Knowledge base search, user profile loading, and database operations. |

---

## Agent Sessions

`AgentSession` is the main orchestrator of our app. It is responsible for collecting user input, managing the voice pipeline, invoking the LLM, sending output back to the user, and emitting events for observability.

### Example of a Simple Agent Session

```python
from livekit.agents import AgentSession, Agent, inference, room_io, TurnHandlingOptions
from livekit.plugins import noise_cancellation

session = AgentSession(
    stt="deepgram/nova-3:en",
    llm="openai/chat-latest",
    tts="cartesia/sonic-3:9626c31c-bec5-4cca-baa8-f8ba9e84c8bc",
    turn_handling=TurnHandlingOptions(
        turn_detection=inference.TurnDetector(),
    ),
)

await session.start(
    room=ctx.room,
    agent=Agent(instructions="You are a helpful voice AI assistant."),
    room_options=room_io.RoomOptions(
        audio_input=room_io.AudioInputOptions(
            noise_cancellation=noise_cancellation.BVC(),
        ),
    ),
)
```

### Lifecycle of a Session

- **Initialization**: The session is setting up. During initialization, no audio or video processing occurs.
- **Starting**: The session is started using the `start()` method. It initializes agent activity tracking and begins forwarding audio and video frames.
- **Running**: The session is actively processing user input and generating agent responses. In this phase, our agent can transfer control to other agents.
- **Closing**: The session emits a close event and resets internal state.

We can monitor agent state changes via the `agent_state_changed` event.

### Events

| Event | Description |
| --- | --- |
| [agent_state_changed](https://docs.livekit.io/reference/agents/events/) | Emitted when the agent's state changes (for example, from listening to thinking or speaking). |
| [user_state_changed](https://docs.livekit.io/reference/agents/events/) | Emitted when the user's state changes (for example, from listening to speaking). |
| [user_input_transcribed](https://docs.livekit.io/reference/agents/events/) | Emitted when user speech is transcribed to text. |
| [conversation_item_added](https://docs.livekit.io/reference/agents/events/) | Emitted when a message is added to the conversation history. |
| [close](https://docs.livekit.io/reference/agents/events/) | Emitted when the session closes, either gracefully or due to an error. |

### Turn Detection

We can configure turn detection mode and turn handling with the `turn_handling` parameter of `AgentSession`.

### User Interaction

- **user_away_timeout**: Time in seconds of silence before the framework sets the user state to away.
- **min_consecutive_speech_delay**: Minimum delay in seconds between consecutive agent utterances.

When neither the user nor the agent has spoken for the configured duration, the framework sets the user state to away and emits a `user_state_changed` event.

### Handling Inactive Users

- Set `user_away_timeout` to control how long silence lasts before the user is marked away.
- Listen for `user_state_changed` and check for `new_state == "away"` to trigger check-in logic.
- Cancel any pending check-in task if the user speaks and the state changes back to speaking or listening.
- Call `session.shutdown()` to end the session.

#### Example

```python
import asyncio
from livekit.agents import (
    Agent, AgentSession, JobContext, UserStateChangedEvent, inference,
)
from livekit.plugins import silero

# ctx is the JobContext from the entrypoint function
session = AgentSession(
    vad=silero.VAD.load(),
    stt=inference.STT("deepgram/nova-3"),
    llm=inference.LLM("openai/gpt-4.1-mini"),
    tts=inference.TTS("cartesia/sonic-3"),
    user_away_timeout=12.5,  # seconds of silence before "away"
)

inactivity_task: asyncio.Task | None = None

async def check_if_user_present():
    # Prompt the user a few times, then end the session
    for _ in range(3):
        await session.generate_reply(
            instructions="The user has been inactive. Politely check if the user is still present."
        )
        await asyncio.sleep(10)
    session.shutdown()

@session.on("user_state_changed")
def on_user_state_changed(ev: UserStateChangedEvent):
    global inactivity_task
    if ev.new_state == "away":
        inactivity_task = asyncio.create_task(check_if_user_present())
        return
    # User came back (speaking, listening, etc.) — cancel the check-in
    if inactivity_task is not None:
        inactivity_task.cancel()
        inactivity_task = None

await session.start(
    agent=Agent(instructions="You are a helpful assistant."),
    room=ctx.room,
)
```

### Optional Entrypoint Parameters

We have the following optional parameters available when we define our entrypoint function using the `rtc_session` decorator:

- **agent_name**: Name of agent for agent dispatch.
- **type**: Determines when a new instance of the agent is created — for each room or for each publisher in a room.
- **on_session_end**: Callback function to be called when the session ends.
- **on_request**: Callback function to be called when a new request is received.

### RoomIO

`RoomIO` serves as a bridge between the agent session and the LiveKit room. When an `AgentSession` is initiated, it automatically creates a `RoomIO` object that enables all room participants to subscribe to available audio tracks.

### Linked Participant

In a session, an agent interacts with a specific linked participant. By default, the linked participant is the first participant to join a room.

To set the linked participant manually:

- Pass the participant identity to the `RoomIO` constructor when creating the session.
- Call `RoomIO.set_participant()` within a session to change the linked participant dynamically.

To identify the linked participant:

```python
await session.start(
    # ... agent, room, room_options, etc.
)

participant = session.room_io.linked_participant
```

### Room Options

Configure how the agent interacts with room participants using `RoomOptions`.

| Component | Description | Use cases |
| --- | --- | --- |
| [Input options](https://docs.livekit.io/agents/logic/sessions/) | Configure input options for text, audio, and video. | Enable noise cancellation, pre-connect audio, or configure additional audio input options. Enable video input, add a callback function for text input, or disable text input entirely. |
| [Output options](https://docs.livekit.io/agents/logic/sessions/) | Configure output options for text and audio. | Set transcription options, disable audio output, or set audio output sample rate, number of channels, and track options. |
| [Participant management](https://docs.livekit.io/agents/logic/sessions/) | Configure participant management options. | Configure the types of participants an agent can interact with and set the linked participant for the session. |
| [Clean up options](https://docs.livekit.io/agents/logic/sessions/) | Configure options for cleaning up the session and room. | Close the session when the linked participant leaves or automatically delete the room on session end. |

#### Input Options

- **Text input options**: To enable or turn off text input, set `RoomOptions.text_input` to `True` or `False`.
- **Audio input options**: To enable or turn off audio input, set `RoomOptions.audio_input` to `True` or `False`.
- **Video input options**: Set `RoomOptions.video_input` to `True` or `False`.

#### Output Options

Set `RoomOptions.text_output` to `True` or `False`.

#### Participant Management

We use the following parameters to configure which types of participants our agent can interact with:

- **participant_kinds**: List of participant types accepted for auto subscription. By default, includes `SIP`, `STANDARD`, and `CONNECTOR` participants.

#### Clean Up Options

**Close when a participant leaves**: By default, the session automatically closes when the linked participant leaves the room for any of the following reasons:

- **CLIENT_INITIATED**: User initiated the disconnect.
- **ROOM_DELETED**: Delete room API was called.
- **USER_REJECTED**: Call was rejected by the user (for example, the line was busy).

**Delete room when session ends**: Optionally configure the session to automatically delete the room once it ends, rather than leaving an empty room behind.

---

## Chat Context

Reference: [Chat context](https://docs.livekit.io/agents/logic/chat-context/)

### Why It Matters

`ChatContext` is the conversation history sent to the LLM on every turn. It holds an ordered list of items — messages and events like agent handoffs — that together define what the model knows about the current conversation. Because each agent and task maintains its own `chat_ctx`, controlling what's inside it is how you control what the model "remembers," what it forgets, and how much it costs to run each turn (smaller, well-pruned contexts mean lower token usage and faster responses). This matters most in multi-agent workflows, where you have to decide what history should follow a user across a handoff and what should be left behind.

### How It Works

By default, a new agent or task starts with an empty context, but you can initialize it at construction time, modify it during turns, or pass it across handoffs.

**Accessing the context**: Within an agent or task, the current context is available as `self.chat_ctx`. The complete conversation history across all agents in a session is available on `session.history`.

**Structure**: `ChatContext` exposes an `items` list. Each item has a `type` field:

| Type | Description |
| --- | --- |
| `message` | A conversation turn with a `role` (`system`, `user`, or `assistant`) and `content` (text, images, or instructions). |
| `function_call` | A tool invocation requested by the LLM. |
| `function_call_output` | The result returned from a tool call. |
| `agent_handoff` | Added automatically when control transfers between agents. |
| `agent_config_update` | Records a change to the agent's instructions or tools (Python only). |

### Core Operations

**Creating a context** — build a context and add messages directly:

```python
from livekit.agents import ChatContext

chat_ctx = ChatContext()
chat_ctx.add_message(role="system", content="You are a helpful assistant.")
chat_ctx.add_message(role="user", content="Hello!")
```

**Copying a context** — use `copy()` to create a snapshot that can be passed to another agent or modified independently. By default, `copy()` includes everything; filter it with options like `exclude_instructions`, `exclude_function_call`, `exclude_handoff`, `exclude_empty_message`, and `exclude_config_update`:

```python
# Copy everything
full_copy = self.chat_ctx.copy()

# Copy only user/assistant turns, without tool calls
turns_only = self.chat_ctx.copy(exclude_instructions=True, exclude_function_call=True)
```

**Truncating a context** — `truncate()` reduces a context to the most recent *n* items. It always preserves system instructions even if they fall outside the item window, and strips leading function call items to avoid orphaned tool results. Useful for passing only the tail of a long conversation to the next agent:

```python
recent = self.chat_ctx.copy().truncate(max_items=6)
```

**Merging contexts** — `merge()` combines items from another context into the current one, deduplicating by item ID and maintaining chronological order. Useful after parallel tasks when you need to reunify their conversation histories:

```python
primary_ctx.merge(other_ctx)

# Merge without carrying over tool calls
primary_ctx.merge(other_ctx, exclude_function_call=True)
```

### Common Patterns

**Initialize with user data** — load user-specific context before the session starts and pass it to the agent constructor. This is the recommended way to personalize the agent without a round-trip to the LLM:

```python
initial_ctx = ChatContext()
initial_ctx.add_message(role="assistant", content=f"The user's name is {user_name}.")

await session.start(
    room=ctx.room,
    agent=MyAgent(chat_ctx=initial_ctx),
)
```

**Modifying context during a turn** — override the `on_user_turn_completed` node to inject additional context before the LLM generates its reply. Messages added here apply to the current turn only; call `update_chat_ctx` to persist them beyond the turn:

```python
from livekit.agents import ChatContext, ChatMessage

async def on_user_turn_completed(
    self, turn_ctx: ChatContext, new_message: ChatMessage,
) -> None:
    extra = await fetch_relevant_data(new_message.text_content)
    turn_ctx.add_message(role="assistant", content=extra)
    await self.update_chat_ctx(turn_ctx)  # persist beyond this turn
```

**Passing context during handoffs** — pass the current context to the next agent to preserve conversation history. Use `exclude_instructions=True` to avoid forwarding the previous agent's system prompt:

```python
return NextAgent(chat_ctx=self.chat_ctx.copy(exclude_instructions=True))
```

For long conversations, summarize the context before passing it on to reduce token cost.

**Adding images and video frames** — message content can include images alongside text by passing a list of text and `ImageContent` items to `add_message`. Live video frames can also be injected during a conversation turn.

**Custom context for `generate_reply()`** — pass a modified `ChatContext` to `generate_reply()` to fully control the context for a single reply. This replaces the agent's session-level context for that reply only, useful for excluding messages, injecting one-off context, or overriding instructions:

```python
ctx = session.current_agent.chat_ctx.copy()
await session.generate_reply(chat_ctx=ctx)
```

**Standalone LLM usage** — `ChatContext` also works outside of agents and sessions. Pass it directly to an LLM's `chat()` method for background tasks, preprocessing, or any workflow that needs LLM output without the voice pipeline.

---

## Tasks & Task Groups

Reference: [Tasks & task groups](https://docs.livekit.io/agents/logic/tasks/)

### Why It Matters

Tasks are focused, reusable units that perform a specific objective and return a typed result. They run inside an agent and take control of the session only until their goal is achieved, which lets you carve a guided, structured sub-conversation (collecting an address, confirming consent, capturing payment details) out of a larger free-flowing conversation without writing a brand-new agent for every small step. For multi-step flows, `TaskGroup` executes an ordered sequence of tasks while letting users return to earlier steps for corrections — useful when a real conversation needs backtracking ("actually, change my email") without losing the overall structure.

Reach for tasks whenever you want a discrete, completable action that returns a typed result: qualifying a lead, collecting patient intake information, running a follow-up survey, gathering booking details, or obtaining recording consent at the start of a call.

### How It Works

#### Defining a Task

Define a task by extending the `AgentTask` class and specifying a result type using generics. Use `on_enter` to begin the task's interaction with the user, and call `complete` with a result when finished. Tasks have full support for tools, similar to an agent:

```python
from livekit.agents import AgentTask, function_tool

class CollectConsent(AgentTask[bool]):
    def __init__(self, chat_ctx=None):
        super().__init__(
            instructions="""
            Ask for recording consent and get a clear yes or no answer.
            Be polite and professional.
            """,
            chat_ctx=chat_ctx,
        )

    async def on_enter(self) -> None:
        await self.session.generate_reply(
            instructions="""
            Briefly introduce yourself, then ask for permission to record the call for quality assurance and training purposes.
            Make it clear that they can decline.
            """
        )

    @function_tool
    async def consent_given(self) -> None:
        """Use this when the user gives consent to record."""
        self.complete(True)

    @function_tool
    async def consent_denied(self) -> None:
        """Use this when the user denies consent to record."""
        self.complete(False)
```

#### Running a Task

A task must be created within the context of an active `Agent`, and runs automatically when created. It takes control of the session until it returns a result; awaiting the task gives you that result.

A task can only be awaited from one of three call sites:

- **`on_enter`**: Runs the task as the agent becomes active. Useful for deterministic setup steps.
- **`on_exit`**: Runs the task as the agent becomes inactive. Useful for wrap-up steps before a handoff or session end.
- **A tool function body**: The tool instantiates and awaits the task. The LLM decides when to invoke the tool, so delegation happens mid-conversation.

Awaiting an `AgentTask` outside these call sites raises a `RuntimeError`.

```python
from livekit import api
from livekit.agents import Agent, function_tool, get_job_context

class CustomerServiceAgent(Agent):
    def __init__(self):
        super().__init__(instructions="You are a friendly customer service representative.")

    async def on_enter(self) -> None:
        if await CollectConsent(chat_ctx=self.chat_ctx):
            await self.session.generate_reply(instructions="Offer your assistance to the user.")
        else:
            await self.session.generate_reply(instructions="Inform the user that you are unable to proceed and will end the call.")
            job_ctx = get_job_context()
            await job_ctx.api.room.delete_room(api.DeleteRoomRequest(room=job_ctx.room.name))
```

#### Passing Conversation History to a Task

By default, a task starts with an empty chat context. To include the parent agent's conversation history, pass `chat_ctx` to the task constructor. Use `exclude_instructions=True` so the task's own instructions take effect instead of the parent's system prompt:

```python
class GetContactInfoTask(AgentTask[ContactInfoResult]):
    def __init__(self, chat_ctx=None):
        super().__init__(
            instructions="Collect the user's name, email address, and phone number.",
            chat_ctx=chat_ctx,
        )

class CustomerServiceAgent(Agent):
    @function_tool()
    async def collect_contact_info(self):
        """Collect the user's contact information."""
        result = await GetContactInfoTask(
            chat_ctx=self.chat_ctx.copy(exclude_instructions=True)
        )
        return f"Recorded contact info for {result.name}."
```

#### Task Results

Use any result type you want; for complex results, use a custom dataclass:

```python
from dataclasses import dataclass

@dataclass
class ContactInfoResult:
    name: str
    email_address: str
    phone_number: str

class GetContactInfoTask(AgentTask[ContactInfoResult]):
    # ....
```

#### Unordered Collection Within Tasks

A single task can collect multiple pieces of information in any order. For example, recording a candidate's strengths, weaknesses, and work style as the user answers in whatever order they choose, completing the task only once all required fields are filled.

### Task Group

**Note**: `TaskGroup` is currently an experimental feature and the API might change in a future release.

Task groups let you build complex, user-friendly workflows that mirror real conversational behavior — where users might need to revisit or correct earlier steps without losing context. They're designed as ordered, multi-step flows broken into discrete tasks, with built-in regression support for safely moving backward. `TaskGroup` supports task chaining, allowing tasks to call or re-enter other tasks dynamically while maintaining overall flow order. All tasks in the group share the same conversation context, and when the group finishes, the summarized context can be passed back to the controlling agent.

**Configuration options**:

- **summarize_chat_ctx** (default `true`): Whether to summarize the interactions within the `TaskGroup` into one message and merge into the main context.
- **chat_ctx**: The shared chat context within the `TaskGroup`. Pass the current chat context to ensure conversational continuity.
- **return_exceptions** (default `false`): Controls error handling when a sub-task raises an unhandled exception. `True` adds the exception to results and continues; `False` propagates immediately and stops.
- **on_task_completed**: An async callback invoked after each sub-task completes successfully, receiving a `TaskCompletedEvent` (with `agent_task`, `task_id`, and `result`).

**Basic usage** — initialize a `TaskGroup` and add tasks in the order they should be executed:

```python
from livekit.agents.beta.workflows import GetEmailTask, TaskGroup

# Create and configure TaskGroup with the current agent's chat context
chat_ctx = self.chat_ctx
task_group = TaskGroup(chat_ctx=chat_ctx)

# Add tasks using lambda factories
task_group.add(
    lambda: GetEmailTask(),
    id="get_email_task",
    description="Collects the user's email"
)
task_group.add(
    lambda: GetCommuteTask(),
    id="get_commute_task",
    description="Records the user's commute flexibility"
)

# Execute the task group
results = await task_group  # Returns TaskGroupResult object
task_results = results.task_results
```

The `TaskGroup.add()` method takes a task factory and an options object:

- **Task factory**: A callable (typically a lambda) that returns a task instance. This allows tasks to be reinitialized with the same arguments when revisited.
- **id**: A string identifier for the task used to access results.
- **description**: A string description that helps the LLM understand when to regress to this task.

**Task completion callbacks** — run custom logic after each task completes using `on_task_completed`, which receives the completed task's ID, instance, and result.

**Early exit from a task group** — avoid calling `session.shutdown()` directly from `on_task_completed` since the group is still iterating its task stack at that point. Instead, raise a custom exception from the callback and catch it where you await the task group. With the default `return_exceptions=False`, `TaskGroup` propagates the exception to the awaiting code, letting you run cleanup logic safely once the group has stopped iterating.

`TaskGroup` uses this same exception-based mechanism internally to handle regression — when the LLM requests to revisit an earlier task, the active task is completed with an internal exception that the group catches and uses to reorder the task stack.

### Best Practices for Testing Task Groups

- Add a short delay (for example, `await asyncio.sleep(0.5)`) between `session.start()` and the first `session.run()` call in Python tests, since `TaskGroup` temporarily sets `llm=None` during task transitions.
- Use built-in assertion helpers like `contains_function_call` (which parse JSON arguments and support partial-dict matching) rather than manually parsing `item.arguments`.
- Initialize `userdata`/`userData` when tasks depend on it, or accessing it unset raises an error.
- Don't assert on startup output (from `on_enter`); structure tests to assert agent responses to user input instead.
- Avoid awaiting speech playout inside `onEnter()` when triggered from a tool, since the tool call remains active until `onEnter()` returns — call `generate_reply()` without awaiting it in that case.
- Prefer `containsFunctionCall()` over `nextEvent()` for resilience, since an LLM might not call a task's completion tool on the very first turn.

---

## Workflows

Reference: [Workflows](https://docs.livekit.io/agents/logic/workflows/)

### Why It Matters

The LiveKit Agents framework lets you build sophisticated voice AI apps with multiple personas, conversation phases, or specialized capabilities using agents, handoffs, and tasks. Rather than cramming every behavior into one giant system prompt, workflows make multi-step systems explicit and predictable: agents manage ongoing conversational control, tasks encapsulate discrete operations, tools execute side effects and enable handoffs, and task groups coordinate ordered multi-step flows with regression support. This produces a testable, maintainable execution model instead of one sprawling, hard-to-debug prompt.

### Core Constructs

An **agent session** is the main orchestrator of your voice AI app and can be composed of one or more agents. Each construct plays a distinct role:

- **Agents** hold long-lived control of a session. They define instructions, reasoning behavior, and tools, and can transfer control to another agent when different rules or capabilities are required.
- **Tools** are user-defined functions callable by the model. They let the agent perform actions beyond generative text, such as reading from or writing to external systems. Tool invocations are model-driven — the LLM chooses to call them based on context — and tools can also trigger agent handoffs.
- **Tasks** are short-lived units of work that run to completion and return a typed result. Unlike agents, tasks don't persist; they take temporary control only while executing.
- **Task groups** run sequences of tasks for multi-step operations, allowing users to revisit earlier steps and sharing conversation context across the group.

### How It Works: Choosing a Pattern

Start with a single agent and a small set of tools. A single agent can handle multi-step flows by updating its instructions or changing available tools between conversation phases. Split workflows into more constructs only when you hit a concrete limitation:

- **Instruction bloat**: The system prompt becomes large enough that the model starts to underperform on its primary task.
- **Conflicting tool access**: Different phases of the conversation require different tools or permissions.
- **Multi-turn data collection**: A workflow step requires its own LLM loop to gather and validate structured input over several turns.
- **Backtracking**: Users need to revisit earlier steps to correct previously provided information.

| Pattern | Session control | Context | Latency cost | Correction handling | Best for |
| --- | --- | --- | --- | --- | --- |
| **Single agent + tools** | One agent stays in control for the entire session. | Full conversation context is retained. | None. | Manual. Re-prompt or re-ask. | Simple flows with few tools and no distinct conversation phases. |
| **Supervisor pattern** | One agent stays in control; tasks take temporary control and return a typed result. | Supervisor keeps full context. Tasks receive a scoped copy. | Minimal. Task runs within the same session. | Re-run the task from the supervisor. | One agent coordinates focused, reusable sub-operations such as data collection or verification. |
| **Agent handoffs** | Control transfers fully to a new agent. The original agent doesn't participate afterward. | Explicit: pass `chat_ctx`, summarize, or start fresh. | Handoff overhead per transition. | Manual. Hand off back to the previous agent. | Distinct roles, model specialization, or permission boundaries between conversation phases. |
| **Task groups** | TaskGroup orchestrates an ordered sequence of tasks. | Shared within the group. Summarized on completion. | Minimal. Sequential within the same session. | Built-in. Users can regress to earlier completed steps. | Ordered multi-step data collection where users might need to revisit earlier steps. |

These patterns aren't mutually exclusive — different phases of a conversation can use different patterns, and they can be combined (for example, an intake supervisor handing off to a billing agent that has its own supervisor pattern).

### Example: Booking Flow

Consider an appointment agent that handles booking, rescheduling, and cancellation. All three intents share an initial lookup step. A single agent with tools works well here if the instructions stay concise and the tools don't conflict.

When the intents diverge — booking needs multi-turn address collection, rescheduling needs calendar-specific tools, cancellation requires a separate consent flow — the supervisor pattern is a better fit, since one supervisor can route to focused specialist tasks while staying in control of the session (handling mid-conversation intent changes if a user starts booking but decides to reschedule). If each intent also requires a strict ordered sequence of sub-steps with backtracking, a task group within the supervisor is appropriate. If the flow ends with a payment step needing different instructions, tools, and access controls, an agent handoff to a dedicated billing agent is appropriate.

### Best Practices

- Create separate agents when you need distinct reasoning behavior or tool access.
- Use tasks for discrete operations that must complete before continuing the conversation.
- Expose external actions through tools with clear purpose and meaningful return values.
- Plan how conversation context is preserved or reset across agents — some transitions need full continuity, others benefit from a clean slate.
- Use a task group for ordered multi-step processes that might need to revisit earlier steps.
- Build workflows incrementally and add tests/evals to verify tool, task, and agent behavior.
- Design for user experience: announce handoffs, preserve relevant context to avoid repetition, and handle correction paths cleanly.

---

## Supervisor Pattern

Reference: [Supervisor pattern](https://docs.livekit.io/agents/logic/supervisor-pattern/)

### Why It Matters

The supervisor pattern keeps a single agent in long-lived control of a session and routes discrete work to specialist tasks. The supervisor decides when each task runs, integrates the result, and continues the conversation — each task is independent, with its own instructions, tools, and LLM loop. This is the pattern of choice when one agent should remain aware of the full conversation while delegating focused operations such as collecting structured information, running verification steps, or retrieving external data, instead of either cramming everything into one prompt or fully handing off (and losing) conversational control.

### How It Works

The pattern has three parts:

- **The supervisor**: A long-lived `Agent` whose instructions define available specialists, when to invoke each one, and how to interpret their results.
- **The specialists**: One or more `AgentTask` instances, each with focused instructions, its own tools, and a typed result.
- **The delegation surface**: The mechanism used to start a specialist — most commonly a function tool on the supervisor, where the tool body instantiates and awaits a task, then returns its result to the LLM. Lifecycle hooks (`on_enter`/`on_exit`) can also trigger tasks at deterministic points.

Tools and tasks are distinct: a tool is regular code that can perform any operation (call an API, write to a database, start a task), while a task is a sub-conversation with its own LLM loop that can in turn call tools.

### When to Use the Supervisor Pattern

- **A single agent with tools** if one set of instructions and tools is sufficient.
- **The supervisor pattern** when one agent should remain in control while delegating discrete operations to focused, reusable tasks.
- **Agent handoffs** when one agent's role is complete and another should take over with different instructions or tools, with the original agent not participating afterward.
- **Task groups** for ordered, multi-step delegation where users might need to revisit earlier steps.

### Designing a Supervisor

**Sizing tasks** — decide what belongs in a task versus a tool versus the supervisor itself:

- **Tool**: Single deterministic operations that don't require LLM reasoning (fetching a record by ID, sending an email, a computation).
- **Task**: Focused sub-conversations that require reasoning and might span multiple turns (collecting a structured address, handling consent, verifying identity).
- **Supervisor**: The conversational frame and routing logic — domain-specific reasoning shouldn't live here.

A useful guideline: if the model needs to ask clarifying questions, the work belongs in a task; if it's a single function call with arguments, it belongs in a tool.

**Writing supervisor instructions** — name each specialist tool explicitly and describe when to use it, since routing behavior depends heavily on these descriptions. Instructions should also define how to interpret each result.

**Validating results** — treat task results as untrusted input until validated by the supervisor. Even though task results are typed, value-level validation is still required, and you should define a recovery path for errors or unexpected outputs.

### Example: Routing Between Specialist Tasks

```python
from dataclasses import dataclass
from livekit.agents import Agent, AgentTask, function_tool

@dataclass
class OrderLookupResult:
    order_id: str
    status: str

@dataclass
class AddressUpdateResult:
    address: str

class LookupOrderTask(AgentTask[OrderLookupResult]):
    def __init__(self, chat_ctx=None) -> None:
        super().__init__(
            instructions=(
                "Ask the customer for their order number. "
                "If they don't have one, ask them to check their email."
            ),
            chat_ctx=chat_ctx,
        )

    async def on_enter(self) -> None:
        await self.session.generate_reply(
            instructions="Ask for the order number."
        )

    @function_tool()
    async def order_number_collected(self, order_id: str) -> None:
        """Call when the customer has provided their order number."""
        self.complete(OrderLookupResult(order_id=order_id, status="shipped"))

class UpdateAddressTask(AgentTask[AddressUpdateResult]):
    def __init__(self, chat_ctx=None) -> None:
        super().__init__(
            instructions=(
                "Collect the customer's new shipping address: street, city, "
                "state, and zip code. Read it back to confirm before completing."
            ),
            chat_ctx=chat_ctx,
        )

    async def on_enter(self) -> None:
        await self.session.generate_reply(
            instructions="Ask for the new shipping address."
        )

    @function_tool()
    async def address_confirmed(self, address: str) -> None:
        """Call once the customer has confirmed their new address."""
        self.complete(AddressUpdateResult(address=address))

class CustomerServiceAgent(Agent):
    def __init__(self) -> None:
        super().__init__(
            instructions=(
                "You are a customer service representative. Greet the caller "
                "and ask how you can help. Route their request:\n"
                "- For questions about order status, call lookup_order.\n"
                "- For shipping address changes, call update_address.\n"
                "After a tool returns, summarize the outcome and ask whether "
                "the caller needs anything else."
            ),
        )

    @function_tool()
    async def lookup_order(self) -> str:
        """Use when the customer wants to check the status of an order."""
        result = await LookupOrderTask(
            chat_ctx=self.chat_ctx.copy(exclude_instructions=True),
        )
        return f"Order {result.order_id} is {result.status}."

    @function_tool()
    async def update_address(self) -> str:
        """Use when the customer wants to change their shipping address."""
        result = await UpdateAddressTask(
            chat_ctx=self.chat_ctx.copy(exclude_instructions=True),
        )
        return f"Updated shipping address to: {result.address}."
```

The supervisor's instructions name each specialist and describe when to invoke it; the LLM uses those descriptions to route incoming requests. Each task starts when its tool is called, takes over the session until it calls `complete(...)`, and returns its typed result to the supervisor for the rest of the conversation.

### Best Practices

- Keep specialist tasks focused: one objective per task with a clear typed result. Split tasks that grow in scope.
- Describe routing precisely in supervisor instructions — ambiguity leads to misrouting.
- Validate task results before continuing the conversation.
- Test the supervisor and each task independently.
- Pass conversation context only when needed; omitting it when not required improves performance and reduces noise.

---

## Tool Definition & Use

Reference: [Tool definition & use](https://docs.livekit.io/agents/build/tools/)

### Why It Matters

Tools let your agent do more than generate text: extend agent context, create interactive experiences, and overcome LLM limitations by calling external functions. Within a tool, you can generate agent speech, call methods on the frontend using RPC, hand off control to another agent as part of a workflow, store and retrieve session data, call external APIs, or do anything else regular code can do. Tools can also run in the background so the agent can keep talking while long-running work completes.

### How It Works

#### Tool Types

- **Function tools**: Tools defined as functions within your agent's codebase, callable by the LLM.
- **Provider tools**: Tools provided by a specific model provider (OpenAI, Gemini, xAI, etc.) and executed internally by the provider's model server — for example, web search, code execution, or file search. These can be mixed with function tools by passing them to the `tools` parameter on your `Agent` (Python only).

```python
from livekit.plugins import openai  # replace with any supported provider

agent = MyAgent(
    llm=openai.responses.LLM(model="gpt-4.1"),
    tools=[openai.tools.WebSearch()],  # replace with any supported tool
)
```

#### Defining a Function Tool

Add tools to your agent class with the `@function_tool` decorator:

```python
from typing import Any
from livekit.agents import function_tool, Agent, RunContext

class MyAgent(Agent):
    @function_tool()
    async def lookup_weather(
        self,
        context: RunContext,
        location: str,
    ) -> dict[str, Any]:
        """Look up weather information for a given location.

        Args:
            location: The location to look up weather information for.
        """
        return {"weather": "sunny", "temperature_f": 70}
```

A good tool definition is key to reliable tool use from the LLM: be specific about what the tool does, when it should or should not be used, what the arguments are for, and what type of return value to expect.

**Tool flags** control when a tool is available. `IGNORE_ON_ENTER` excludes a tool from any `generate_reply` calls made inside the agent's `on_enter` method — useful for a "confirm address" tool that shouldn't be callable before the user has actually provided an address.

#### Arguments and Return Values

Tool arguments are automatically inferred from the function signature; parameter names and type hints are sent to the LLM as part of the tool schema. The tool's return value is automatically converted to a string before being sent to the LLM, which then generates a new reply or additional tool calls based on it. Returning `None` (or nothing) completes the tool silently without requiring a reply.

You can also use the return value to trigger a handoff to a different `Agent` within a workflow — return a tuple of the `Agent` instance and an optional result message.

#### RunContext

Tools include support for a special `context` argument (`RunContext`), which gives access to the current `session`, `function_call`, `speech_handle`, and `userdata`.

#### Using Speech in Tool Calls

You can generate agent speech from inside a tool using `session.say()` or `session.generate_reply()`, useful for telling the user "this may take a moment" before a slow operation runs:

```python
@function_tool()
async def process_order(self, context: RunContext, order_id: str):
    """Process an order and notify the user."""
    await self.session.generate_reply(
        instructions=f"Processing order {order_id}. This may take a moment."
    )
    result = await process_order_internal(order_id)
    return result
```

#### Interruptions

By default, tools can be interrupted if the user speaks; when interrupted, the tool is removed from history and any result is ignored. If your tool takes external actions that can't be rolled back (placing an order, sending a message), disable interruptions by calling `run_ctx.disallow_interruptions()` at the start of the tool. For tools that take more than a few seconds, consider an async/background tool instead so the agent can keep talking while the tool works.

#### Calling HTTP APIs

Most tools call an external HTTP API. Best practices:

- **Disable interruptions for mutating calls** (placing an order, scheduling an appointment) so an interruption can't leave the operation partially complete.
- **Raise `ToolError` on failure** so the LLM gets an informative error message instead of the tool crashing.
- **Store credentials in environment variables**, not hard-coded in the tool.
- **Set explicit timeouts** so a slow external service doesn't block the agent indefinitely.

```python
import aiohttp
from livekit.agents import Agent, RunContext, function_tool
from livekit.agents.llm import ToolError

class StockAgent(Agent):
    @function_tool()
    async def get_stock_price(self, context: RunContext, symbol: str) -> dict:
        """Get the current stock price for a given ticker symbol.

        Args:
            symbol: The stock ticker symbol, for example AAPL or GOOGL.
        """
        url = f"https://livekit-stock-api.vercel.app/api/quote?symbol={symbol}"
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                if response.status != 200:
                    raise ToolError(f"Could not fetch stock price for {symbol}.")
                data = await response.json()
                return {
                    "symbol": data["symbol"],
                    "price": data["price"],
                    "volume": data["volume"],
                    "latest_trading_day": data["latestTradingDay"],
                }
```

#### Adding Tools Dynamically

You can share a tool between multiple agents by defining it outside their class and passing it to each. Use `agent.update_tools()` to replace all tools on an agent at runtime — useful for swapping the tool set as a conversation moves between phases.

#### Creating Tools Programmatically and From Raw Schema

`function_tool` can be called as a regular function (not just a decorator) to compose tools dynamically — useful when loading tool definitions from a database or an MCP server, or when giving the agent multiple narrow tools backed by the same underlying code. You can also create tools directly from a raw JSON function-calling schema using the `raw_schema` parameter, useful for integrating with existing function definitions.

#### Error Handling

Raise the `ToolError` exception to return a descriptive error to the LLM in place of a normal response, so the agent can inform the user gracefully rather than crashing:

```python
@function_tool()
async def lookup_weather(self, context: RunContext, location: str) -> dict:
    if location == "mars":
        raise ToolError("This location is coming soon. Please join our mailing list to stay updated.")
    else:
        return {"weather": "sunny", "temperature_f": 70}
```

#### Related Capabilities

- **Toolsets**: Bundle related tools under a single ID to add or remove them as a group (the built-in `MCPToolset` is built on this primitive).
- **Async tools**: Run long-running tools in the background so the agent can keep talking (Python only).
- **Model Context Protocol (MCP)**: Expose tools from MCP servers to your agent (Python only).
- **Forwarding to the frontend**: Fulfill tool calls via RPC from the client.

---

## Pipeline Nodes & Hooks

Reference: [Pipeline nodes & hooks](https://docs.livekit.io/agents/logic/nodes/)

### Why It Matters

You can fully customize your agent's behavior at multiple **nodes** in the processing path — a node is a point where one process transitions to another. This is how you swap in a custom STT/LLM/TTS provider without a plugin, generate a custom greeting, strip filler words from a transcript before it reaches the LLM, fix pronunciation in the LLM's output before it reaches TTS, or update a UI when the agent or user finishes speaking. Without nodes, you'd be limited to whatever behavior the default pipeline gives you; nodes let you intercept and modify data at each stage.

### How It Works

Override the relevant method within a custom `Agent` subclass to customize behavior at a specific node. Call `Agent.default.<node-name>()` to fall back to default behavior for the parts you don't want to change:

```python
async def stt_node(self, audio: AsyncIterable[rtc.AudioFrame], model_settings: ModelSettings) -> Optional[AsyncIterable[stt.SpeechEvent]]:
    # insert custom before STT processing here
    events = Agent.default.stt_node(self, audio, model_settings)
    # insert custom after STT processing here
    return events
```

### Lifecycle Hooks

- **`on_enter()`**: Called after the agent becomes the active agent in a session. Common use: generate a greeting.
- **`on_exit()`**: Called before the agent gives control to another agent in the same session, as part of a workflow. Use it to save data, say goodbye, or clean up.
- **`on_user_turn_completed()`**: Called when the user's turn has ended, before the agent's reply. The node receives `turn_ctx` (the full chat context up to but not including the latest user message) and `new_message` (the user's latest message). Use it for RAG — retrieving context relevant to the newest message and injecting it into the chat context — or to modify, cancel, or block the agent's reply (for example, raising `StopResponse` for push-to-talk interfaces where there's no real content to respond to). Changes made via `turn_ctx.add_message()` are scoped to the current turn unless you call `update_chat_ctx` to persist them; editing `new_message.content` directly persists going forward.
- **`on_user_turn_exceeded()`**: Called when the user has been speaking long enough to exceed a configured user turn limit (`max_words` or `max_duration`). The default implementation politely cuts in with a non-interruptible reply; override it to customize the behavior, for example with a fixed "Sorry to jump in" message via `say()`.

**Fast pre-response pattern**: Use `on_user_turn_completed` together with a smaller, faster model to speak a short filler phrase ("let me think about that") while the main reply is still generating, calling `say()` without awaiting it so the two run concurrently and reduce the perceived response gap.

### STT-LLM-TTS Pipeline Nodes

These nodes are only available for STT-LLM-TTS pipeline models (not realtime models):

- **`stt_node()`**: Transcribes audio frames into speech events, converting user audio into text for the LLM. Override it for custom audio pre-processing, additional buffering, alternative STT strategies, or post-processing of the transcribed text.
- **`llm_node()`**: Performs inference on the chat context and produces the agent's response or tool calls (as plain text or `llm.ChatChunk` objects). Override it to customize how the LLM is invoked, modify the chat context before inference, adjust tool-call handling, or implement a custom LLM provider without a plugin. In Python, you can also yield a `FlushSentinel` to flush partial text to TTS immediately — for example, speaking "One moment while I look that up" while a slow tool call is still running, instead of leaving the user in silence.
- **`tts_node()`**: Synthesizes audio from text segments, converting LLM output into speech. Override it for custom text chunking, a custom TTS engine, pronunciation rules, volume adjustment, or other audio processing (such as time-stretching the audio to speed up or slow down playback without changing pitch).

### Realtime Model Nodes

- **`realtime_audio_output_node()`**: Called when a realtime model outputs speech, letting you modify the audio before it's sent to the user (for example, adjusting volume).

### Transcription Node

- **`transcription_node()`**: Part of the forwarding path for agent transcriptions; used to clean up formatting, fix punctuation, strip unwanted characters, or access TTS-aligned transcription timestamps. By default it simply passes the transcription through to the designated output.

---

## Turn Detection & Interruptions

Reference: [Turn detection & interruptions](https://docs.livekit.io/agents/logic/turns/)

### Why It Matters

Turn detection is the process of determining when a user begins or ends their "turn" in a conversation, letting the agent know when to start listening and when to respond. Effective turn detection and interruption management is essential to a natural-feeling voice AI experience — get it wrong and the agent talks over the user, responds too slowly, or fails to notice when it's been interrupted. Most techniques rely on voice activity detection (VAD) to find periods of silence, with heuristics or a meaning-aware model layered on top to figure out when a sentence or thought is actually complete (as opposed to just a pause mid-sentence).

### How It Works

#### Turn Detection Modes

| Mode | When to use it |
| --- | --- |
| **Turn detector model** | Recommended for most agents, and the default. Predicts end of turn from the meaning of speech, on top of VAD. |
| **Realtime models** | When using a realtime LLM (OpenAI Realtime API, Gemini Live API), rely on its built-in server-side detection or pair it with the turn detector model. |
| **VAD only** | When you need minimal latency, or support for a spoken language the turn detector model doesn't cover. |
| **STT endpointing** | When you're already using an STT with its own turn detection (AssemblyAI, Deepgram Flux). |
| **Manual turn control** | For push-to-talk or fully explicit control over turn boundaries. |

**Turn detector model** (the default and recommended option) is enabled automatically by `AgentSession`, so most agents need no configuration at all:

```python
from livekit.agents import AgentSession, TurnHandlingOptions, inference

session = AgentSession(
    turn_handling=TurnHandlingOptions(
        turn_detection=inference.TurnDetector(),
    ),
    # ... stt, tts, llm, etc.
)
```

**Realtime models** include their own built-in turn detection based on VAD and other techniques. When you use a realtime model with server-side turn detection, the model itself decides when the user is interrupting, and most `InterruptionOptions` fields don't apply — tune interruption on the model itself (for example, the OpenAI Realtime API exposes `threshold`, `prefix_padding_ms`, and `silence_duration_ms` on its server VAD object).

**VAD only** is for minimal-latency cases or languages the turn detector model doesn't support: set `turn_detection="vad"` and use the Silero VAD plugin.

**STT endpointing** relies on an STT provider's own turn detection (set `turn_detection="stt"`); you should still provide a VAD plugin for responsive interruption handling, since STT-only endpointing makes the agent less responsive to interruptions.

**Manual turn control** disables automatic detection entirely (`turn_detection="manual"`), giving you explicit control via `session.interrupt()`, `session.clear_user_turn()`, and `session.commit_user_turn()` — the standard way to build a push-to-talk interface using RPC methods the frontend calls when the user presses and releases a talk button:

```python
session = AgentSession(
    turn_handling=TurnHandlingOptions(
        turn_detection="manual",
    ),
    # ... stt, tts, llm, etc.
)

session.input.set_audio_enabled(False)

@ctx.room.local_participant.register_rpc_method("start_turn")
async def start_turn(data: rtc.RpcInvocationData):
    session.interrupt()
    session.clear_user_turn()
    session.input.set_audio_enabled(True)

@ctx.room.local_participant.register_rpc_method("end_turn")
async def end_turn(data: rtc.RpcInvocationData):
    session.input.set_audio_enabled(False)
    session.commit_user_turn()

@ctx.room.local_participant.register_rpc_method("cancel_turn")
async def cancel_turn(data: rtc.RpcInvocationData):
    session.input.set_audio_enabled(False)
    session.clear_user_turn()
```

In Python, `commit_user_turn()` returns an `asyncio.Future[str]` you can await to capture the transcript, and accepts `skip_reply=True` to commit and transcribe without generating a reply.

**Reducing background noise**: Enhanced noise cancellation (available on LiveKit Cloud) improves the quality of both turn detection and STT by reducing background noise; add it via room options when starting the session.

#### Interruptions

The framework pauses the agent's speech whenever it detects user speech in the input audio. The user can interrupt at any time, either by speaking or via `session.interrupt()`. When interrupted, the agent stops speaking and automatically truncates its conversation history to include only the portion of speech the user actually heard.

**Interruption mode** is controlled by two key settings: `enabled` (whether the agent can be interrupted at all) and `mode`, which determines how interruptions are detected:

- `"adaptive"`: Adaptive interruption handling — the default mode on LiveKit Cloud with most STT providers. Analyzes audio signals to distinguish true interruptions from conversational backchanneling (like "mm-hmm") rather than using a fixed threshold.
- `"vad"`: Uses VAD alone, based on speech start/stop cues.

**False interruptions**: Sometimes the framework detects speech audio and interrupts the agent, but the transcription comes up empty — no actual words were spoken. By default, the agent resumes speaking from where it left off after such a false interruption; this is configurable via `resume_false_interruption` and `false_interruption_timeout`.

**Additional interruption options** include `discard_audio_if_uninterruptible` (drop buffered audio if the agent is speaking and can't be interrupted), `min_duration` (minimum speech duration to register as an interruption), and `min_words` (minimum word count, useful for requiring actual speech content before triggering an interruption).

#### User Turn Limit

User turn limits cap how long a user can speak before the agent steps in — useful for voicebot scenarios where a caller might monopolize the turn (long-form callers, voicemail greetings, users reading off a list). Unlike interruptions (user-initiated), user turn limits are agent-initiated. Configure `max_words` and/or `max_duration` in `turn_handling.user_turn_limit`; both default to disabled. When a threshold is crossed, the framework calls `on_user_turn_exceeded`, whose default implementation politely cuts in with a non-interruptible reply.

#### Session Events

`AgentSession` emits events you can subscribe to for monitoring conversation flow:

- **Interruption events**: `user_interruption_detected` (with timestamp and probability) and `agent_false_interruption`.
- **Turn-taking events**: `user_state_changed` (`speaking`, `listening`, `away`) and `agent_state_changed` (`initializing`, `idle`, `listening`, `thinking`, `speaking`).

```python
from livekit.agents import UserStateChangedEvent, AgentStateChangedEvent

@session.on("user_state_changed")
def on_user_state_changed(ev: UserStateChangedEvent):
    if ev.new_state == "speaking":
        print("User started speaking")
    elif ev.new_state == "listening":
        print("User stopped speaking")
    elif ev.new_state == "away":
        print("User is not present (e.g. disconnected)")

@session.on("agent_state_changed")
def on_agent_state_changed(ev: AgentStateChangedEvent):
    if ev.new_state == "thinking":
        print("Agent is processing user input and generating a response")
    elif ev.new_state == "speaking":
        print("Agent started speaking")
```

---

## Fallback Strategies

Reference: [Fallback strategies](https://docs.livekit.io/agents/logic/fallback-strategies/)

### Why It Matters

In realtime voice conversations, a model API failure (connection failure, timeout, HTTP error, mid-stream disconnect) can leave the agent unable to continue mid-call. Fallback strategies let you define backup providers that automatically take over when the primary provider fails, so a single provider outage doesn't translate into a dropped or broken conversation for the user. When a fallback triggers, `AgentSession` emits an error event you can use to log the failure or notify the user.

### How It Works

LiveKit provides two fallback mechanisms:

| Feature | Inference Fallback Adapter | Agent Fallback Adapter |
| --- | --- | --- |
| Supported model types | STT, TTS | STT, TTS, LLM |
| Where fallback runs | Server-side in LiveKit Inference service | In your agent process |

Both adapters trigger on any error from the primary provider, automatically resubmit the failed request to backup providers, mark the failed provider unhealthy and stop sending it requests, and continue using the backups while periodically probing the failed provider in the background to restore it once it responds successfully again.

#### Inference Fallback Adapter

If you use LiveKit Inference, configure fallback models directly with the `fallback` parameter on `inference.STT` and `inference.TTS`. Fallback logic runs server-side, so your agent code doesn't need to manage retries or health checks itself:

```python
from livekit.agents import AgentSession, inference

session = AgentSession(
    stt=inference.STT(
        model="deepgram/nova-3",
        language="en",
        fallback=[
            {"model": "assemblyai/universal-streaming"},
        ],
    ),
    tts=inference.TTS(
        model="cartesia/sonic-3",
        voice="9626c31c-bec5-4cca-baa8-f8ba9e84c8bc",
        language="en",
        fallback=[
            {
                "model": "inworld/inworld-tts-1.5-max",
                "voice": "Ashley",
            },
        ],
    ),
    # ... llm, etc.
)
```

The top-level model is the primary; models in `fallback` are tried in order if the primary fails. If the primary fails partway through streaming a response, the service switches models and restarts the request from the beginning, and only stops trying once every configured model has failed or the client disconnects.

If you use custom voices, TTS fallback across providers is automatic, since each cloned voice is cloned to more than one provider.

#### Agent Fallback Adapter

Use this when you need LLM fallback support, or when connecting to providers not available through LiveKit Inference. Fallback logic runs directly in your agent process using plugins:

```python
from livekit.agents import llm, stt, tts
from livekit.plugins import assemblyai, cartesia, deepgram, inworld, openai

session = AgentSession(
    stt=stt.FallbackAdapter(
        [
            deepgram.STT(),
            assemblyai.STT(),
        ]
    ),
    llm=llm.FallbackAdapter(
        [
            openai.responses.LLM(model="gpt-4o"),
            openai.LLM.with_azure(model="gpt-4o", ...),
        ]
    ),
    tts=tts.FallbackAdapter(
        [
            cartesia.TTS(...),
            inworld.TTS(...),
        ]
    ),
)
```

The first instance in each list is the primary; subsequent ones are tried in order if it fails.

**Behavior** — the Agent Fallback Adapter applies partial output guards so a mid-stream switch doesn't disrupt output the user has already started receiving:

- **STT**: No partial output guard — switches to the next provider on any error.
- **TTS**: If audio has already been pushed to the speaker, the adapter does not switch providers mid-utterance; fallback is skipped and the partial audio plays through.
- **LLM**: If text or tool calls have already been streamed to the user, the adapter raises the error rather than restarting the response with a different model, unless you set `retry_on_chunk_sent=True` to explicitly allow mid-stream fallback.

When a provider recovers after a failure, the Agent Fallback Adapter emits an availability-changed event (`stt_availability_changed`, `llm_availability_changed`, or `tts_availability_changed`) so you can observe the recovery from your agent code.
