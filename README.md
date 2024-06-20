# OmniAI::OpenAI

[![CircleCI](https://circleci.com/gh/ksylvest/omniai-openai.svg?style=svg)](https://circleci.com/gh/ksylvest/omniai-openai)

An OpenAI implementation of the [OmniAI](https://github.com/ksylvest/omniai) APIs.

## Installation

```sh
gem install omniai-openai
```

## Usage

### Client

A client is setup as follows if `ENV['OPENAI_API_KEY']` exists:

```ruby
client = OmniAI::OpenAI::Client.new
```

A client may also be passed the following options:

- `api_key` (required - default is `ENV['OPENAI_API_KEY']`)
- `organization` (optional)
- `project` (optional)
- `host` (optional)

### Configuration

Global configuration is supported for the following options:

```ruby
OmniAI::OpenAI.configure do |config|
  config.api_key = 'sk-...' # default: ENV['OPENAI_API_KEY']
  config.organization = '...' # default: ENV['OPENAI_ORGANIZATION']
  config.project = '...' # default: ENV['OPENAI_PROJECT']
  config.host = '...' # default: 'https://api.openai.com'
end
```

### Chat

A chat completion is generated by passing in prompts using any a variety of formats:

```ruby
completion = client.chat.completion('Tell me a joke!')
completion.choice.message.content # 'Why did the chicken cross the road? To get to the other side.'
```

```ruby
completion = client.chat.completion({
  role: OmniAI::Chat::Role::USER,
  content: 'Is it wise to jump off a bridge?'
})
completion.choice.message.content # 'No.'
```

```ruby
completion = client.chat.completion([
  {
    role: OmniAI::Chat::Role::SYSTEM,
    content: 'You are a helpful assistant.'
  },
  'What is the capital of Canada?',
])
completion.choice.message.content # 'The capital of Canada is Ottawa.'
```

#### Model

`model` takes an optional string (default is `gtp-4o`):

```ruby
completion = client.chat.completion('How fast is a cheetah?', model: OmniAI::OpenAI::Chat::Model::GPT_3_5_TURBO)
completion.choice.message.content # 'A cheetah can reach speeds over 100 km/h.'
```

[OpenAI API Reference `model`](https://platform.openai.com/docs/api-reference/chat/create#chat-create-model)

#### Temperature

`temperature` takes an optional float between `0.0` and `2.0` (defaults is `0.7`):

```ruby
completion = client.chat.completion('Pick a number between 1 and 5', temperature: 2.0)
completion.choice.message.content # '3'
```

[OpenAI API Reference `temperature`](https://platform.openai.com/docs/api-reference/chat/create#chat-create-temperature)

#### Stream

`stream` takes an optional a proc to stream responses in real-time chunks instead of waiting for a complete response:

```ruby
stream = proc do |chunk|
  print(chunk.choice.delta.content) # 'Better', 'three', 'hours', ...
end
client.chat.completion('Be poetic.', stream:)
```

[OpenAI API Reference `stream`](https://platform.openai.com/docs/api-reference/chat/create#chat-create-stream)

#### Format

`format` takes an optional symbol (`:json`) and that setes the `response_format` to `json_object`:

```ruby
completion = client.chat.completion([
  { role: OmniAI::Chat::Role::SYSTEM, content: OmniAI::Chat::JSON_PROMPT },
  { role: OmniAI::Chat::Role::USER, content: 'What is the name of the drummer for the Beatles?' }
], format: :json)
JSON.parse(completion.choice.message.content) # { "name": "Ringo" }
```

[OpenAI API Reference `response_format`](https://platform.openai.com/docs/api-reference/chat/create#chat-create-stream)

> When using JSON mode, you must also instruct the model to produce JSON yourself via a system or user message.

### Transcribe

A transcription is generated by passing in a path to a file:

```ruby
transcription = client.transcribe(file.path)
transcription.text # '...'
```

#### Prompt

`prompt` is optional and can provide additional context for transcribing:

```ruby
transcription = client.transcribe(file.path, prompt: '')
transcription.text # '...'
```

[OpenAI API Reference `prompt`](https://platform.openai.com/docs/api-reference/audio/createTranscription#audio-createtranscription-prompt)

#### Format

`format` is optional and supports `json`, `text`, `srt` or `vtt`:

```ruby
transcription = client.transcribe(file.path, format: OmniAI::Transcribe::Format::TEXT)
transcription.text # '...'
```

[OpenAI API Reference `response_format`](https://platform.openai.com/docs/api-reference/audio/createTranscription#audio-createtranscription-response_format)

#### Language

`language` is optional and may improve accuracy and latency:

```ruby
transcription = client.transcribe(file.path, language: OmniAI::Transcribe::Language::SPANISH)
transcription.text
```

[OpenAI API Reference `language`](https://platform.openai.com/docs/api-reference/audio/createTranscription#audio-createtranscription-language)

#### Temperature

`temperature` is optional and must be between 0.0 (more deterministic) and 1.0 (less deterministic):

```ruby
transcription = client.transcribe(file.path, temperature: 0.2)
transcription.text
```

[OpenAI API Reference `temperature`](https://platform.openai.com/docs/api-reference/audio/createTranscription#audio-createtranscription-temperature)

### Speak

Speech can be generated by passing text with a block:

```ruby
File.open('example.ogg', 'wb') do |file|
  client.speak('How can a clam cram in a clean cream can?') do |chunk|
    file << chunk
  end
end
```

If a block is not provided then a tempfile is returned:

```ruby
tempfile = client.speak('Can you can a can as a canner can can a can?')
tempfile.close
tempfile.unlink
```

#### Voice

`voice` is optional and must be one of the supported voices:

```ruby
client.speak('She sells seashells by the seashore.', voice: OmniAI::OpenAI::Speak::Voice::SHIMMER)
```

[OpenAI API Reference `voice`](https://platform.openai.com/docs/api-reference/audio/createSpeech#audio-createspeech-voice)

#### Model

`model` is optional and must be either `tts-1` or `tts-1-hd` (default):

```ruby
client.speak('I saw a kitten eating chicken in the kitchen.', format: OmniAI::OpenAI::Speak::Model::TTS_1)
```

[OpenAI API Refernce `model`](https://platform.openai.com/docs/api-reference/audio/createSpeech#audio-createspeech-model)

#### Speed

`speed` is optional and must be between 0.25 and 0.40:

```ruby
client.speak('How much wood would a woodchuck chuck if a woodchuck could chuck wood?', speed: 4.0)
```

[OmniAI API Reference `speed`](https://platform.openai.com/docs/api-reference/audio/createSpeech#audio-createspeech-speed)

#### Format

`format` is optional and supports `MP3` (default), `OPUS`, `AAC`, `FLAC`, `WAV` or `PCM`:

```ruby
client.speak('A pessemistic pest exists amidst us.', format: OmniAI::OpenAI::Speak::Format::FLAC)
```

[OpenAI API Reference `format`](https://platform.openai.com/docs/api-reference/audio/createSpeech#audio-createspeech-response_format)
