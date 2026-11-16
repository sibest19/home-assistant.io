---
title: "Evaluate"
action: ai_task.evaluate
domain: ai_task
response: required
description: "Evaluates questions about supplied state and attachments using AI."
related_actions:
  - ai_task.generate_data
---

The **Evaluate** action answers questions about text, JSON data, or attachments. Use it to estimate whether something is true, choose among alternatives, or assess ordered levels. It returns probabilities and answers that you can use in later steps of an automation or script.

You need an AI task entity that supports evaluation. This action does not support targets: set **Entity ID** in the action, or configure a [preferred evaluation entity](/integrations/ai_task/#choosing-a-preferred-evaluation-entity). Supply **State**, **Attachments**, or both.

{% include actions/ui_header.md %}

To evaluate questions from an automation or a script:

1. Go to {% my automations title="**Settings** > **Automations & scenes**" %}.
2. Open an automation or script, or create one.
3. In the **Then do** section, select **Add action**.
4. Search for and select **Evaluate**.
5. Enter a **Task name** and select an **Entity ID** that supports evaluation, unless you have set a preferred evaluation entity.
6. Enter the **State** to evaluate, select **Attachments**, or supply both.
7. In **Questions**, enter one or more named questions. For a yes/no question, use the example below. See [Question format](#question-format) for other question types.
8. In **Response variable**, enter a name to store the answers, such as `evaluation`.
9. Select **Save**.

For example, enter the following in **Questions** to ask whether a package has arrived:

```yaml
delivered:
  type: noul
  instructions: "Has the package arrived?"
```

### Options in the UI

{% options_ui %}
Task name:
  description: A name that identifies the task.
  required: true
Entity ID:
  description: The AI task entity to use. If omitted, the preferred evaluation entity is used. An entity configured only for data or image generation cannot run this action.
  required: false
State:
  description: The text or JSON data to evaluate. Required when no attachments are supplied.
  required: false
Questions:
  description: Named questions to evaluate against the same state and attachments. Each question has a type and instructions. Choice and score questions also have criteria.
  required: true
Attachments:
  description: Files to evaluate, such as camera snapshots or local media. Required when no state is supplied. Supported file types depend on the selected entity and model.
  required: false
{% endoptions_ui %}

{% include actions/yaml_header.md %}

In YAML, refer to this action as `ai_task.evaluate`. Set `entity_id` inside `data`, rather than `target`. Set `response_variable` at the action level to store the answers.

### Options in YAML

{% options_yaml %}
task_name:
  description: A name that identifies the task.
  required: true
  type: string
entity_id:
  description: The AI task entity to use. If omitted, the preferred evaluation entity is used. If neither is set, the action fails.
  required: false
  type: string
state:
  description: The text or JSON data to evaluate. It can be a string, number, boolean, list, or mapping. You must supply a non-null state, attachments, or both.
  required: false
  type: any
questions:
  description: A nonempty mapping from question IDs to question definitions. See the question format below.
  required: true
  type: map
attachments:
  description: >
    A list of attachments from the
    [Media selector](/docs/blueprint/selectors/#media-selector).
    Each item contains a media_content_id and can include a media_content_type.
    Supported file types depend on the selected entity and model.
  required: false
  type: list
{% endoptions_yaml %}

### Question format

Every question has a nonempty ID, a `type`, and nonempty `instructions` describing what to evaluate. All questions in one call use the same state and attachments.

- `noul`: Estimate the probability that the answer is yes. No `criteria` field is used.
- `choice`: Choose among at least two alternatives. Set `criteria` to a mapping from nonempty alternative IDs to nonempty descriptions.
- `score`: Assess at least two ordered levels. Set `criteria` to a list of nonempty descriptions, in order from the lowest level to the highest. The first level has index 0, the next has index 1, and so on.

## Response data

{% include actions/response.md %}

The response contains an `answers` mapping with exactly one entry for each question ID. Each entry contains its `type` and the following fields:

- `noul`: A `noul` value from 0 to 1, where 0 means no and 1 means yes.
- `choice`: A `choice` containing the selected alternative ID, and a `probabilities` mapping for all alternative IDs. The selected alternative has the highest probability; any tied alternative can be selected.
- `score`: A `score` containing the probability-weighted average of the level indexes, and a `probabilities` list in the same order as `criteria`. The score can be fractional.

All probabilities are between 0 and 1. For choice and score answers, the probabilities sum to 1, allowing for small numerical rounding differences. Home Assistant rejects responses that do not match this format.

For example, with the levels "No immediate action needed" and "Collect it now", the following answer assigns a probability of 0.8 to the first level and 0.2 to the second. The score is 0.2: `0 × 0.8 + 1 × 0.2`.

{% example %}
output: |
  answers:
    urgency:
      type: score
      score: 0.2
      probabilities:
        - 0.8
        - 0.2
{% endexample %}

With a response variable named `evaluation`, the score is available as `evaluation.answers.urgency.score`.

## Good to know

- Evaluation support and supported attachment types depend on the selected entity and model.
- Attachments must resolve to local media. Camera snapshots are supported through the [Camera](/integrations/camera/) integration.
- A successful evaluation updates the AI task entity's state to the time it was used.
- The returned probabilities are AI estimates and can be incorrect.

{% include actions/try_it.md %}

{% include actions/more_examples.md %}

### Automation: Assess a delivery update

When a sensor containing a delivery message changes, evaluate that message with all three question types. Replace the sensor and evaluation entity IDs with your own.

- **Trigger**: State changed
  - **Entity**: Delivery status (`sensor.delivery_status`)
- **Action**: Evaluate
  - **Task name**: Delivery assessment
  - **Entity ID**: Your evaluation entity
  - **State**: Enter {% raw %}`{{ states('sensor.delivery_status') }}`{% endraw %} to supply the sensor's current value. Replace `sensor.delivery_status` with your sensor's entity ID.
  - **Questions**: The questions shown below
  - **Response variable**: `evaluation`

{% details "YAML example for evaluating a delivery message" %}

{% example %}
automation: |
  alias: "Assess a delivery update"
  triggers:
    - trigger: state
      entity_id: sensor.delivery_status
  actions:
    - action: ai_task.evaluate
      data:
        task_name: "Delivery assessment"
        entity_id: ai_task.evaluation
        state: "{{ states('sensor.delivery_status') }}"
        questions:
          delivered:
            type: noul
            instructions: "Has the package arrived?"
          location:
            type: choice
            instructions: "Where was the package left?"
            criteria:
              door: "At the front door"
              other: "Somewhere else"
          urgency:
            type: score
            instructions: "How urgently does the package need collecting?"
            criteria:
              - "No immediate action needed"
              - "Collect it now"
      response_variable: evaluation
{% endexample %}

{% enddetails %}

### Automation: Check a camera snapshot when the doorbell rings

When the doorbell rings, evaluate whether a package is visible. Use an evaluation entity that supports image attachments. Replace the doorbell, camera, and evaluation entity IDs with your own.

- **Trigger**: State changed
  - **Entity**: Doorbell (`binary_sensor.doorbell`)
  - **To**: On
- **Action**: Evaluate
  - **Task name**: Check for a package
  - **Entity ID**: Your evaluation entity
  - **Attachments**: Front door camera (`camera.front_door`)
  - **Questions**: A `noul` question asking whether a package is visible
  - **Response variable**: `package_check`

{% details "YAML example for checking a camera snapshot" %}

{% example %}
automation: |
  alias: "Check for a package when the doorbell rings"
  triggers:
    - trigger: state
      entity_id: binary_sensor.doorbell
      to: "on"
  actions:
    - action: ai_task.evaluate
      data:
        task_name: "Check for a package"
        entity_id: ai_task.evaluation
        attachments:
          - media_content_id: media-source://camera/camera.front_door
            media_content_type: image/jpeg
        questions:
          package:
            type: noul
            instructions: "Is a package visible?"
      response_variable: package_check
{% endexample %}

{% enddetails %}

{% include actions/stuck.md %}

{% include actions/related.md %}
