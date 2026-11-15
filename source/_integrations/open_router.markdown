---
description: Instructions on how to use OpenRouter as a conversation agent and for AI tasks
ha_release: 2025.8
ha_domain: open_router
---

The [OpenRouter](https://openrouter.ai/) {% term integration %} allows you to use the OpenRouter API as a conversation agent and for [AI tasks](/integrations/ai_task/) in Home Assistant.

This integration provides a way to interact with a wide range of AI models available on OpenRouter, while billing is handled by OpenRouter.
You can even use your own third-party (like OpenAI) API key.

{% include integrations/config_flow.md %}

## Generate an API Key

The API key is used to authenticate requests to OpenRouter. To generate an API key take the following steps:

- Log in to [OpenRouter](https://openrouter.ai/) or sign up for an account.
- Go to the **API Keys** section in your account settings.
- To generate a new key, select **Create API Key**.
- Give the key a name, and be sure to set up billing limits.

## Conversation agent options

When you add or reconfigure a conversation agent, you can set the following options:

{% configuration_basic %}
Model:
  description: The OpenRouter model the agent uses. See the [OpenRouter models](https://openrouter.ai/models) page for details.
Instructions:
  description: Instructions for the AI on how it should respond to your requests. It is written using [Home Assistant Templating](/docs/templating/).
AI tools:
  description: "**Device control** only accesses the entities you [expose](/voice_control/voice_remote_expose_devices/) to voice assistants. **Home Assistant** accesses everything and only works for administrators, not voice assistant devices."
Reasoning:
  description: "Whether the model reasons before it answers, and how much. **Auto** (the default) uses the model's own default. **Off** turns reasoning off, which usually makes answers faster. **On** turns reasoning on at the model's default level. **Minimal**, **Low**, **Medium**, **High**, **X-High**, and **Max** set the reasoning effort: higher levels can improve answers to complex requests, but take longer and cost more. Not every model supports every setting. Models without reasoning support only accept **Auto**, models that always reason cannot be set to **Off**, and only the effort levels a model offers are accepted. Routers such as **Auto Router** accept every setting except **Off**. If the model does not support your choice, an error is shown when you save. See [reasoning tokens](https://openrouter.ai/docs/guides/best-practices/reasoning-tokens) for details."
{% endconfiguration_basic %}

## Configuring an evaluation task

1. Go to {% my integrations title="**Settings** > **Devices & services**" %} and select **OpenRouter**.
2. Select **Add AI task evaluation**.
3. Select a **Model** and submit the form. The list only includes models that support the OpenRouter Decisions API.

This creates an AI task entity for evaluation, separate from entities that generate data or images. Select a model that supports the types of input you want to evaluate.

## Supported functionality

### Generating data with AI

The OpenRouter integration allows you to generate data using AI models available on OpenRouter. You can use this functionality in automations, scripts, or directly in the Home Assistant UI.

### Generating images with AI

If you choose a model that can create images when you add an AI task, you can also use it to generate images. Use the [`ai_task.generate_image`](/actions/ai_task.generate_image/) action in your automations and scripts. For generation tasks, models that cannot create images only support generating data.

### Evaluating questions with AI

Use the [**Evaluate**](/actions/ai_task.evaluate/) (`ai_task.evaluate`) action in automations and scripts with your evaluation task entity. See the action documentation for [question types and criteria](/actions/ai_task.evaluate/#question-format), input fields, and response data.

Image and PDF support depend on the selected model. Models that accept only text cannot evaluate attachments.

## Removing the integration

This integration follows standard integration removal, no extra steps are required.

{% include integrations/remove_device_service.md %}
