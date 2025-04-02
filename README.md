# Gemini AI

A Ruby Gem for interacting with [Gemini](https://deepmind.google/technologies/gemini/) through [Vertex AI](https://cloud.google.com/vertex-ai), [Generative Language API](https://ai.google.dev/api/rest), or [AI Studio](https://makersuite.google.com), Google's generative AI services.

![The logo shows a gemstone split into red and blue halves, symbolizing Ruby programming and Gemini AI. It's surrounded by a circuit-like design on a dark blue backdrop.](https://raw.githubusercontent.com/gbaptista/assets/main/gemini-ai/ruby-gemini-ai.png)

> _This Gem is designed to provide low-level access to Gemini, enabling people to build abstractions on top of it. If you are interested in more high-level abstractions or more user-friendly tools, you may want to consider [Nano Bots](https://github.com/icebaker/ruby-nano-bots) 💎 🤖._

## TL;DR and Quick Start

```ruby
gem 'gemini-ai', '~> 4.3.0'
```

```ruby
require 'gemini-ai'

# With an API key
client = Gemini.new(
  credentials: {
    service: 'generative-language-api',
    api_key: ENV['GOOGLE_API_KEY']
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With a Service Account Credentials File
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    file_path: 'google-credentials.json',
    region: 'us-east4'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With the Service Account Credentials File contents
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    file_contents: File.read('google-credentials.json'),
    # file_contents: ENV['GOOGLE_CREDENTIALS_FILE_CONTENTS'],
    region: 'us-east4'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With Application Default Credentials
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    region: 'us-east4'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

result = client.stream_generate_content({
  contents: { role: 'user', parts: { text: 'hi!' } }
})
```

Google now supports a [global endpoint](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/locations#global-endpoint):

```ruby
Gemini.new(credentials: { region: 'global' })
```

Result:
```ruby
[{ 'candidates' =>
   [{ 'content' => {
        'role' => 'model',
        'parts' => [{ 'text' => 'Hello! How may I assist you?' }]
      },
      'finishReason' => 'STOP',
      'safetyRatings' =>
      [{ 'category' => 'HARM_CATEGORY_HARASSMENT', 'probability' => 'NEGLIGIBLE' },
       { 'category' => 'HARM_CATEGORY_HATE_SPEECH', 'probability' => 'NEGLIGIBLE' },
       { 'category' => 'HARM_CATEGORY_SEXUALLY_EXPLICIT', 'probability' => 'NEGLIGIBLE' },
       { 'category' => 'HARM_CATEGORY_DANGEROUS_CONTENT', 'probability' => 'NEGLIGIBLE' }] }],
   'usageMetadata' => {
     'promptTokenCount' => 2,
     'candidatesTokenCount' => 8,
     'totalTokenCount' => 10
   } }]
```

## Index

- [TL;DR and Quick Start](#tldr-and-quick-start)
- [Index](#index)
- [Setup](#setup)
  - [Installing](#installing)
  - [Credentials](#credentials)
    - [Option 1: API Key (Generative Language API)](#option-1-api-key-generative-language-api)
    - [Option 2: Service Account Credentials File (Vertex AI API)](#option-2-service-account-credentials-file-vertex-ai-api)
    - [Option 3: Application Default Credentials (Vertex AI API)](#option-3-application-default-credentials-vertex-ai-api)
    - [Required Data](#required-data)
  - [Custom Version](#custom-version)
  - [Global Endpoint](#global-endpoint)
- [Available Models](#available-models)
- [Usage](#usage)
  - [Client](#client)
  - [Methods](#methods)
    - [Chat](#chat)
      - [stream_generate_content](#stream_generate_content)
        - [Receiving Stream Events](#receiving-stream-events)
        - [Without Events](#without-events)
      - [generate_content](#generate_content)
    - [Embeddings](#embeddings)
      - [predict](#predict)
      - [embed_content](#embed_content)
  - [Modes](#modes)
    - [Text](#text)
    - [Image](#image)
    - [Video](#video)
  - [Streaming vs. Server-Sent Events (SSE)](#streaming-vs-server-sent-events-sse)
    - [Server-Sent Events (SSE) Hang](#server-sent-events-sse-hang)
    - [Non-Streaming](#non-streaming)
  - [Back-and-Forth Conversations](#back-and-forth-conversations)
  - [Safety Settings](#safety-settings)
  - [System Instructions](#system-instructions)
  - [Counting Tokens](#counting-tokens)
  - [JSON Format Responses](#json-format-responses)
    - [JSON Schema](#json-schema)
    - [Models That Support JSON](#models-that-support-json)
  - [Tools (Functions) Calling](#tools-functions-calling)
  - [New Functionalities and APIs](#new-functionalities-and-apis)
  - [Request Options](#request-options)
    - [Adapter](#adapter)
    - [Timeout](#timeout)
  - [Error Handling](#error-handling)
    - [Rescuing](#rescuing)
    - [For Short](#for-short)
    - [Errors](#errors)
- [Development](#development)
  - [Purpose](#purpose)
  - [Publish to RubyGems](#publish-to-rubygems)
  - [Updating the README](#updating-the-readme)
- [Resources and References](#resources-and-references)
- [Disclaimer](#disclaimer)

## Setup

### Installing

```sh
gem install gemini-ai -v 4.3.0
```

```sh
gem 'gemini-ai', '~> 4.3.0'
```

### Credentials

- [Option 1: API Key (Generative Language API)](#option-1-api-key-generative-language-api)
- [Option 2: Service Account Credentials File (Vertex AI API)](#option-2-service-account-credentials-file-vertex-ai-api)
- [Option 3: Application Default Credentials (Vertex AI API)](#option-3-application-default-credentials-vertex-ai-api)
- [Required Data](#required-data)

> ⚠️ DISCLAIMER: Be careful with what you are doing, and never trust others' code related to this. These commands and instructions alter the level of access to your Google Cloud Account, and running them naively can lead to security risks as well as financial risks. People with access to your account can use it to steal data or incur charges. Run these commands at your own responsibility and due diligence; expect no warranties from the contributors of this project.

#### Option 1: API Key (Generative Language API)

You need a [Google Cloud](https://console.cloud.google.com) [_Project_](https://cloud.google.com/resource-manager/docs/creating-managing-projects), and then you can generate an API Key through the Google Cloud Console [here](https://console.cloud.google.com/apis/credentials).

You also need to enable the _Generative Language API_ service in your Google Cloud Console, which can be done [here](https://console.cloud.google.com/apis/library/generativelanguage.googleapis.com).


Alternatively, you can generate an API Key through _Google AI Studio_ [here](https://makersuite.google.com/app/apikey). However, this approach will automatically create a project for you in your Google Cloud Account.

#### Option 2: Service Account Credentials File (Vertex AI API)

You need a [Google Cloud](https://console.cloud.google.com) [_Project_](https://cloud.google.com/resource-manager/docs/creating-managing-projects) and a [_Service Account_](https://cloud.google.com/iam/docs/service-account-overview) to use [Vertex AI](https://cloud.google.com/vertex-ai) API.

After creating them, you need to enable the Vertex AI API for your project by clicking `Enable` here: [Vertex AI API](https://console.cloud.google.com/apis/library/aiplatform.googleapis.com).

You can create credentials for your _Service Account_ [here](https://console.cloud.google.com/apis/credentials), where you will be able to download a JSON file named `google-credentials.json` that should have content similar to this:

```json
{
  "type": "service_account",
  "project_id": "YOUR_PROJECT_ID",
  "private_key_id": "a00...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "PROJECT_ID@PROJECT_ID.iam.gserviceaccount.com",
  "client_id": "000...",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/..."
}
```

You need to have the necessary [policies](https://cloud.google.com/iam/docs/policies) (`roles/aiplatform.user` and possibly `roles/ml.admin`) in place to use the Vertex AI API.

You can add them by navigating to the [IAM Console](https://console.cloud.google.com/iam-admin/iam) and clicking on the _"Edit principal"_ (✏️ pencil icon) next to your _Service Account_.

Alternatively, you can add them through the [gcloud CLI](https://cloud.google.com/sdk/gcloud) as follows:

```sh
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member='serviceAccount:PROJECT_ID@PROJECT_ID.iam.gserviceaccount.com' \
  --role='roles/aiplatform.user'
```

Some people reported having trouble accessing the API, and adding the role `roles/ml.admin` fixed it:

```sh
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member='serviceAccount:PROJECT_ID@PROJECT_ID.iam.gserviceaccount.com' \
  --role='roles/ml.admin'
```

If you are not using a _Service Account_:
```sh
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member='user:YOUR@MAIL.COM' \
  --role='roles/aiplatform.user'

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member='user:YOUR@MAIL.COM' \
  --role='roles/ml.admin'
```

#### Option 3: Application Default Credentials (Vertex AI API)

Similar to [Option 2](#option-2-service-account-credentials-file-vertex-ai-api), but you don't need to download a `google-credentials.json`. [_Application Default Credentials_](https://cloud.google.com/docs/authentication/application-default-credentials) automatically find credentials based on the application environment.

For local development, you can generate your default credentials using the [gcloud CLI](https://cloud.google.com/sdk/gcloud) as follows:

```sh
gcloud auth application-default login
```

For more details about alternative methods and different environments, check the official documentation:
[Set up Application Default Credentials](https://cloud.google.com/docs/authentication/provide-credentials-adc)

#### Required Data

After choosing an option, you should have all the necessary data and access to use Gemini.

**Option 1**, for API Key:

```ruby
{
  service: 'generative-language-api',
  api_key: 'GOOGLE_API_KEY'
}
```

Remember that hardcoding your API key in code is unsafe; it's preferable to use environment variables:

```ruby
{
  service: 'generative-language-api',
  api_key: ENV['GOOGLE_API_KEY']
}
```

**Option 2**: For the Service Account, provide a `google-credentials.json` file and a `REGION`:

```ruby
{
  service: 'vertex-ai-api',
  file_path: 'google-credentials.json',
  region: 'us-east4'
}
```

Alternatively, you can pass the file contents instead of the path:
```ruby
{
  service: 'vertex-ai-api',
  file_contents: File.read('google-credentials.json'),
  region: 'us-east4'
}
```

```ruby
{
  service: 'vertex-ai-api',
  file_contents: ENV['GOOGLE_CREDENTIALS_FILE_CONTENTS'],
  region: 'us-east4'
}
```

**Option 3**: For _Application Default Credentials_, omit both the `api_key` and the `file_path`:

```ruby
{
  service: 'vertex-ai-api',
  region: 'us-east4'
}
```

As of the writing of this README, the following regions support Gemini:
```text
Iowa (us-central1)
Las Vegas, Nevada (us-west4)
Montréal, Canada (northamerica-northeast1)
Northern Virginia (us-east4)
Oregon (us-west1)
Seoul, Korea (asia-northeast3)
Singapore (asia-southeast1)
Tokyo, Japan (asia-northeast1)
```

You can follow here if new regions are available: [Gemini API](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/gemini)

You might want to explicitly set a Google Cloud Project ID, which you can do as follows:

```ruby
{
  service: 'vertex-ai-api',
  project_id: 'PROJECT_ID'
}
```

### Custom Version

By default, the gem uses the `v1` version of the APIs. You may want to use a different version:

```ruby
# With an API key
client = Gemini.new(
  credentials: {
    service: 'generative-language-api',
    api_key: ENV['GOOGLE_API_KEY'],
    version: 'v1beta'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With a Service Account Credentials File
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    file_path: 'google-credentials.json',
    region: 'us-east4',
    version: 'v1beta'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With the Service Account Credentials File contents
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    file_contents: File.read('google-credentials.json'),
    # file_contents: ENV['GOOGLE_CREDENTIALS_FILE_CONTENTS'],
    region: 'us-east4'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With Application Default Credentials
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    region: 'us-east4',
    version: 'v1beta'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)
```

### Global Endpoint

Google now supports a [global endpoint](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/locations#global-endpoint):

```ruby
Gemini.new(credentials: { region: 'global' })
```

## Available Models

These models are accessible to the repository **author** as of June 2025 in the `us-east4` region. Access to models may vary by region, user, and account. All models here are expected to work, if you can access them. This is just a reference of what a "typical" user may expect to have access to right away:

| Model                                    | Vertex AI | Generative Language |
|------------------------------------------|:---------:|:-------------------:|
| gemini-pro-vision                        |    ✅     |          🔒         |
| gemini-pro                               |    ✅     |          ✅         |
| gemini-1.5-pro-preview-0514              |    ✅     |          🔒         |
| gemini-1.5-pro-preview-0409              |    ✅     |          🔒         |
| gemini-1.5-pro                           |    ✅     |          ✅         |
| gemini-1.5-flash-preview-0514            |    ✅     |          🔒         |
| gemini-1.5-flash                         |    ✅     |          ✅         |
| gemini-1.0-pro-vision-latest             |    🔒     |          🔒         |
| gemini-1.0-pro-vision-001                |    ✅     |          🔒         |
| gemini-1.0-pro-vision                    |    ✅     |          🔒         |
| gemini-1.0-pro-latest                    |    🔒     |          ✅         |
| gemini-1.0-pro-002                       |    ✅     |          🔒         |
| gemini-1.0-pro-001                       |    ✅     |          ✅         |
| gemini-1.0-pro                           |    ✅     |          ✅         |
| gemini-ultra                             |    🔒     |          🔒         |
| gemini-1.0-ultra                         |    🔒     |          🔒         |
| gemini-1.0-ultra-001                     |    🔒     |          🔒         |
| text-embedding-preview-0514              |    🔒     |          🔒         |
| text-embedding-preview-0409              |    🔒     |          🔒         |
| text-embedding-004                       |    ✅     |          ✅         |
| embedding-001                            |    🔒     |          ✅         |
| text-multilingual-embedding-002          |    ✅     |          🔒         |
| textembedding-gecko-multilingual@001     |    ✅     |          🔒         |
| textembedding-gecko-multilingual@latest  |    ✅     |          🔒         |
| textembedding-gecko@001                  |    ✅     |          🔒         |
| textembedding-gecko@002                  |    ✅     |          🔒         |
| textembedding-gecko@003                  |    ✅     |          🔒         |
| textembedding-gecko@latest               |    ✅     |          🔒         |

You can follow new models at:

- [Google models](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/models)
  - [Model versions and lifecycle](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/model-versioning)

This is [the code](https://gist.github.com/gbaptista/d7390901293bce81ee12ff4ec5fed62c) used for generating this table that you can use to explore your own access.

## Usage

### Client
Ensure that you have all the [required data](#required-data) for authentication.

Create a new client:
```ruby
require 'gemini-ai'

# With an API key
client = Gemini.new(
  credentials: {
    service: 'generative-language-api',
    api_key: ENV['GOOGLE_API_KEY']
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With a Service Account Credentials File
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    file_path: 'google-credentials.json',
    region: 'us-east4'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With the Service Account Credentials File contents
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    file_contents: File.read('google-credentials.json'),
    # file_contents: ENV['GOOGLE_CREDENTIALS_FILE_CONTENTS'],
    region: 'us-east4'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)

# With Application Default Credentials
client = Gemini.new(
  credentials: {
    service: 'vertex-ai-api',
    region: 'us-east4'
  },
  options: { model: 'gemini-pro', server_sent_events: true }
)
```

### Methods

#### Chat

##### stream_generate_content

###### Receiving Stream Events

Ensure that you have enabled [Server-Sent Events](#streaming-vs-server-sent-events-sse) before using blocks for streaming:

```ruby
client.stream_generate_content(
  { contents: { role: 'user', parts: { text: 'hi!' } } }
) do |event, parsed, raw|
  puts event
end
```

Event:
```ruby
{ 'candidates' =>
  [{ 'content' => {
       'role' => 'model',
       'parts' => [{ 'text' => 'Hello! How may I assist you?' }]
     },
     'finishReason' => 'STOP',
     'safetyRatings' =>
     [{ 'category' => 'HARM_CATEGORY_HARASSMENT', 'probability' => 'NEGLIGIBLE' },
      { 'category' => 'HARM_CATEGORY_HATE_SPEECH', 'probability' => 'NEGLIGIBLE' },
      { 'category' => 'HARM_CATEGORY_SEXUALLY_EXPLICIT', 'probability' => 'NEGLIGIBLE' },
      { 'category' => 'HARM_CATEGORY_DANGEROUS_CONTENT', 'probability' => 'NEGLIGIBLE' }] }],
  'usageMetadata' => {
    'promptTokenCount' => 2,
    'candidatesTokenCount' => 8,
    'totalTokenCount' => 10
  } }
```

###### Without Events

You can use `stream_generate_content` without events:

```ruby
result = client.stream_generate_content(
  { contents: { role: 'user', parts: { text: 'hi!' } } }
)
```

In this case, the result will be an array with all the received events:

```ruby
[{ 'candidates' =>
   [{ 'content' => {
        'role' => 'model',
        'parts' => [{ 'text' => 'Hello! How may I assist you?' }]
      },
      'finishReason' => 'STOP',
      'safetyRatings' =>
      [{ 'category' => 'HARM_CATEGORY_HARASSMENT', 'probability' => 'NEGLIGIBLE' },
       { 'category' => 'HARM_CATEGORY_HATE_SPEECH', 'probability' => 'NEGLIGIBLE' },
       { 'category' => 'HARM_CATEGORY_SEXUALLY_EXPLICIT', 'probability' => 'NEGLIGIBLE' },
       { 'category' => 'HARM_CATEGORY_DANGEROUS_CONTENT', 'probability' => 'NEGLIGIBLE' }] }],
   'usageMetadata' => {
     'promptTokenCount' => 2,
     'candidatesTokenCount' => 8,
     'totalTokenCount' => 10
   } }]
```

You can mix both as well:
```ruby
result = client.stream_generate_content(
  { contents: { role: 'user', parts: { text: 'hi!' } } }
) do |event, parsed, raw|
  puts event
end
```

##### generate_content

```ruby
result = client.generate_content(
  { contents: { role: 'user', parts: { text: 'hi!' } } }
)
```

Result:
```ruby
{ 'candidates' =>
  [{ 'content' => { 'parts' => [{ 'text' => 'Hello! How can I assist you today?' }], 'role' => 'model' },
     'finishReason' => 'STOP',
     'index' => 0,
     'safetyRatings' =>
     [{ 'category' => 'HARM_CAT