<script context="module" lang="ts">
    import { getApiBase, getEndpointCompletions, getEndpointGenerations } from '../../ApiUtil.svelte'
    import { countTokens } from '../../Models.svelte'
    import { countMessageTokens } from '../../Stats.svelte'
    import { globalStorage } from '../../Storage.svelte'
    import type { Chat, Message, Model, ModelDetail } from '../../Types.svelte'
    import { chatRequest, imageRequest } from './request.svelte'
    import { checkModel } from './util.svelte'
    import { encode } from 'gpt-tokenizer'
    import { get } from 'svelte/store'

    const hiddenSettings = {
      startSequence: true,
      stopSequence: true,
      aggressiveStop: true,
      delimiter: true,
      userMessageStart: true,
      userMessageEnd: true,
      assistantMessageStart: true,
      assistantMessageEnd: true,
      systemMessageStart: true,
      systemMessageEnd: true,
      repetitionPenalty: true,
      holdSocket: true
      // leadPrompt: true
    } as any

    const chatModelBase = {
      type: 'chat',
      help: 'Below are the settings that OpenAI allows to be changed for the API calls. See the <a target="_blank" href="https://platform.openai.com/docs/api-reference/chat/create">OpenAI API docs</a> for more details.',
      preFillMerge: (existingContent, newContent) => {
        if (existingContent && !newContent.match(/^('(t|ll|ve|m|d|re)[^a-z]|
          \s|[.,;:(_-{}*^%$#@!?+=~`[\]])/i)) {
          existingContent += ' '
        }
        return existingContent
      },
      request: chatRequest,
      check: checkModel,
      getTokens: (value) => encode(value),
      getEndpoint: (model) => get(globalStorage).openAICompletionEndpoint || (getApiBase() + getEndpointCompletions()),
      hideSetting: (chatId, setting) => !!hiddenSettings[setting.key],
      countMessageTokens: (message: Message, model: Model, chat: Chat) => {
        return countTokens(model, '## ' + message.role + ' ##:\r\n\r\n' + message.content + '\r\n\r\n\r\n')
      },
      countPromptTokens: (prompts: Message[], model: Model, chat: Chat): number => {
        return prompts.reduce((a, m) => {
          a += countMessageTokens(m, model, chat)
          return a
        }, 0) + 3
      }
    } as ModelDetail

    // Pricing constants
    const gpt35 = {
      ...chatModelBase,
      prompt: 0.0000015,
      completion: 0.000002,
      max: 4096
    }
    const gpt3516k = {
      ...chatModelBase,
      prompt: 0.000001,
      completion: 0.0000015,
      max: 16384
    }
    const gpt4 = {
      ...chatModelBase,
      prompt: 0.00003,
      completion: 0.00006,
      max: 8192
    }
    const gpt432k = {
      ...chatModelBase,
      prompt: 0.00006,
      completion: 0.00012,
      max: 32768
    }
    const gpt4128kpreview = {
      ...chatModelBase,
      prompt: 0.00001,
      completion: 0.00003,
      max: 131072
    }

    // New model details
    const o3Detail = {
      ...chatModelBase,
      prompt: 10    / 1_000_000,
      completion: 40    / 1_000_000,
      max: 128 * 1024
    }
    const gpt41Detail = {
      ...chatModelBase,
      prompt: 2     / 1_000_000,
      completion: 8     / 1_000_000,
      max: 1_000_000
    }
    const gpt45Detail = {
      ...chatModelBase,
      prompt: 75    / 1_000_000,
      completion: 150   / 1_000_000,
      max: 128 * 1024
    }

    export const chatModels: Record<string, ModelDetail> = {
      'gpt-3.5-turbo': { ...gpt3516k },
      'gpt-3.5-turbo-0301': { ...gpt35 },
      'gpt-3.5-turbo-0613': { ...gpt35 },
      'gpt-3.5-turbo-1106': { ...gpt3516k },
      'gpt-3.5-turbo-16k': { ...gpt3516k },
      'gpt-3.5-turbo-16k-0613': { ...gpt3516k },
      'gpt-4': { ...gpt4 },
      'gpt-4-0314': { ...gpt4 },
      'gpt-4-0613': { ...gpt4 },
      'gpt-4-1106-preview': { ...gpt4128kpreview },
      'gpt-4-32k': { ...gpt432k },
      'gpt-4-32k-0314': { ...gpt432k },
      'gpt-4-32k-0613': { ...gpt432k },
      // new models
      'o3':      { ...o3Detail },
      'gpt-4.1': { ...gpt41Detail },
      'gpt-4.5': { ...gpt45Detail }
    }

    const imageModelBase = {
      type: 'image',
      prompt: 0.00,
      max: 1000,
      request: imageRequest,
      check: checkModel,
      getTokens: (value) => [0],
      getEndpoint: (model) => getApiBase() + getEndpointGenerations(),
      hideSetting: (chatId, setting) => false
    } as ModelDetail

    export const imageModels: Record<string, ModelDetail> = {
      'dall-e-1024x1024': {
        ...imageModelBase,
        completion: 0.020,
        opt: { size: '1024x1024' }
      },
      'dall-e-512x512': {
        ...imageModelBase,
        completion: 0.018,
        opt: { size: '512x512' }
      },
      'dall-e-256x256': {
        ...imageModelBase,
        type: 'image',
        completion: 0.016,
        opt: { size: '256x256' }
      },
      'dall-e-3-1024x1024': {
        ...imageModelBase,
        type: 'image',
        completion: 0.04,
        opt: { model: 'dall-e-3', size: '1024x1024' }
      },
      'dall-e-3-1024x1792-Portrait': {
        ...imageModelBase,
        type: 'image',
        completion: 0.08,
        opt: { model: 'dall-e-3', size: '1024x1792' }
      },
      'dall-e-3-1792x1024-Landscape': {
        ...imageModelBase,
        type: 'image',
        completion: 0.08,
        opt: { model: 'dall-e-3', size: '1792x1024' }
      },
      'dall-e-3-1024x1024-HD': {
        ...imageModelBase,
        type: 'image',
        completion: 0.08,
        opt: { model: 'dall-e-3', size: '1024x1024', quality: 'hd' }
      },
      'dall-e-3-1024x1792-Portrait-HD': {
        ...imageModelBase,
        type: 'image',
        completion: 0.12,
        opt: { model: 'dall-e-3', size: '1024x1792', quality: 'hd' }
      },
      'dall-e-3-1792x1024-Landscape-HD': {
        ...imageModelBase,
        type: 'image',
        completion: 0.12,
        opt: { model: 'dall-e-3', size: '1792x1024', quality: 'hd' }
      }
    }
</script>
