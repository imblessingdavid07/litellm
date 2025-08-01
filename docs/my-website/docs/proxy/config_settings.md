# All settings

```yaml
environment_variables: {}

model_list:
  - model_name: string
    litellm_params: {}
    model_info:
      id: string
      mode: embedding
      input_cost_per_token: 0
      output_cost_per_token: 0
      max_tokens: 2048
      base_model: gpt-4-1106-preview
      additionalProp1: {}

litellm_settings:
  # Logging/Callback settings
  success_callback: ["langfuse"]  # list of success callbacks
  failure_callback: ["sentry"]  # list of failure callbacks
  callbacks: ["otel"]  # list of callbacks - runs on success and failure
  service_callbacks: ["datadog", "prometheus"]  # logs redis, postgres failures on datadog, prometheus
  turn_off_message_logging: boolean  # prevent the messages and responses from being logged to on your callbacks, but request metadata will still be logged.
  redact_user_api_key_info: boolean  # Redact information about the user api key (hashed token, user_id, team id, etc.), from logs. Currently supported for Langfuse, OpenTelemetry, Logfire, ArizeAI logging.
  langfuse_default_tags: ["cache_hit", "cache_key", "proxy_base_url", "user_api_key_alias", "user_api_key_user_id", "user_api_key_user_email", "user_api_key_team_alias", "semantic-similarity", "proxy_base_url"] # default tags for Langfuse Logging
  
  # Networking settings
  request_timeout: 10 # (int) llm requesttimeout in seconds. Raise Timeout error if call takes longer than 10s. Sets litellm.request_timeout 
  force_ipv4: boolean # If true, litellm will force ipv4 for all LLM requests. Some users have seen httpx ConnectionError when using ipv6 + Anthropic API
  
  set_verbose: boolean # sets litellm.set_verbose=True to view verbose debug logs. DO NOT LEAVE THIS ON IN PRODUCTION
  json_logs: boolean # if true, logs will be in json format

  # Fallbacks, reliability
  default_fallbacks: ["claude-opus"] # set default_fallbacks, in case a specific model group is misconfigured / bad.
  content_policy_fallbacks: [{"gpt-3.5-turbo-small": ["claude-opus"]}] # fallbacks for ContentPolicyErrors
  context_window_fallbacks: [{"gpt-3.5-turbo-small": ["gpt-3.5-turbo-large", "claude-opus"]}] # fallbacks for ContextWindowExceededErrors



  # Caching settings
  cache: true 
  cache_params:        # set cache params for redis
    type: redis        # type of cache to initialize

    # Optional - Redis Settings
    host: "localhost"  # The host address for the Redis cache. Required if type is "redis".
    port: 6379  # The port number for the Redis cache. Required if type is "redis".
    password: "your_password"  # The password for the Redis cache. Required if type is "redis".
    namespace: "litellm.caching.caching" # namespace for redis cache
  
    # Optional - Redis Cluster Settings
    redis_startup_nodes: [{"host": "127.0.0.1", "port": "7001"}] 

    # Optional - Redis Sentinel Settings
    service_name: "mymaster"
    sentinel_nodes: [["localhost", 26379]]

    # Optional - Qdrant Semantic Cache Settings
    qdrant_semantic_cache_embedding_model: openai-embedding # the model should be defined on the model_list
    qdrant_collection_name: test_collection
    qdrant_quantization_config: binary
    similarity_threshold: 0.8   # similarity threshold for semantic cache

    # Optional - S3 Cache Settings
    s3_bucket_name: cache-bucket-litellm   # AWS Bucket Name for S3
    s3_region_name: us-west-2              # AWS Region Name for S3
    s3_aws_access_key_id: os.environ/AWS_ACCESS_KEY_ID  # us os.environ/<variable name> to pass environment variables. This is AWS Access Key ID for S3
    s3_aws_secret_access_key: os.environ/AWS_SECRET_ACCESS_KEY  # AWS Secret Access Key for S3
    s3_endpoint_url: https://s3.amazonaws.com  # [OPTIONAL] S3 endpoint URL, if you want to use Backblaze/cloudflare s3 bucket

    # Common Cache settings
    # Optional - Supported call types for caching
    supported_call_types: ["acompletion", "atext_completion", "aembedding", "atranscription"]
                          # /chat/completions, /completions, /embeddings, /audio/transcriptions
    mode: default_off # if default_off, you need to opt in to caching on a per call basis
    ttl: 600 # ttl for caching
    disable_copilot_system_to_assistant: False  # If false (default), converts all 'system' role messages to 'assistant' for GitHub Copilot compatibility. Set to true to disable this behavior.


callback_settings:
  otel:
    message_logging: boolean  # OTEL logging callback specific settings

general_settings:
  completion_model: string
  disable_spend_logs: boolean  # turn off writing each transaction to the db
  disable_master_key_return: boolean  # turn off returning master key on UI (checked on '/user/info' endpoint)
  disable_retry_on_max_parallel_request_limit_error: boolean  # turn off retries when max parallel request limit is reached
  disable_reset_budget: boolean  # turn off reset budget scheduled task
  disable_adding_master_key_hash_to_db: boolean  # turn off storing master key hash in db, for spend tracking
  enable_jwt_auth: boolean  # allow proxy admin to auth in via jwt tokens with 'litellm_proxy_admin' in claims
  enforce_user_param: boolean  # requires all openai endpoint requests to have a 'user' param
  allowed_routes: ["route1", "route2"]  # list of allowed proxy API routes - a user can access. (currently JWT-Auth only)
  key_management_system: google_kms  # either google_kms or azure_kms
  master_key: string
  maximum_spend_logs_retention_period: 30d # The maximum time to retain spend logs before deletion.
  maximum_spend_logs_retention_interval: 1d # interval in which the spend log cleanup task should run in.

  # Database Settings
  database_url: string
  database_connection_pool_limit: 0  # default 100
  database_connection_timeout: 0  # default 60s
  allow_requests_on_db_unavailable: boolean  # if true, will allow requests that can not connect to the DB to verify Virtual Key to still work 

  custom_auth: string
  max_parallel_requests: 0  # the max parallel requests allowed per deployment 
  global_max_parallel_requests: 0  # the max parallel requests allowed on the proxy all up 
  infer_model_from_keys: true
  background_health_checks: true
  health_check_interval: 300
  alerting: ["slack", "email"]
  alerting_threshold: 0
  use_client_credentials_pass_through_routes: boolean  # use client credentials for all pass through routes like "/vertex-ai", /bedrock/. When this is True Virtual Key auth will not be applied on these endpoints
```

### litellm_settings - Reference

| Name | Type | Description |
|------|------|-------------|
| success_callback | array of strings | List of success callbacks. [Doc Proxy logging callbacks](logging), [Doc Metrics](prometheus) |
| failure_callback | array of strings | List of failure callbacks [Doc Proxy logging callbacks](logging), [Doc Metrics](prometheus) |
| callbacks | array of strings | List of callbacks - runs on success and failure [Doc Proxy logging callbacks](logging), [Doc Metrics](prometheus) |
| service_callbacks | array of strings | System health monitoring - Logs redis, postgres failures on specified services (e.g. datadog, prometheus) [Doc Metrics](prometheus) |
| turn_off_message_logging | boolean | If true, prevents messages and responses from being logged to callbacks, but request metadata will still be logged [Proxy Logging](logging) |
| modify_params | boolean | If true, allows modifying the parameters of the request before it is sent to the LLM provider |
| enable_preview_features | boolean | If true, enables preview features - e.g. Azure O1 Models with streaming support.|
| redact_user_api_key_info | boolean | If true, redacts information about the user api key from logs [Proxy Logging](logging#redacting-userapikeyinfo) |
| langfuse_default_tags | array of strings | Default tags for Langfuse Logging. Use this if you want to control which LiteLLM-specific fields are logged as tags by the LiteLLM proxy. By default LiteLLM Proxy logs no LiteLLM-specific fields as tags. [Further docs](./logging#litellm-specific-tags-on-langfuse---cache_hit-cache_key) |
| set_verbose | boolean | If true, sets litellm.set_verbose=True to view verbose debug logs. DO NOT LEAVE THIS ON IN PRODUCTION |
| json_logs | boolean | If true, logs will be in json format. If you need to store the logs as JSON, just set the `litellm.json_logs = True`. We currently just log the raw POST request from litellm as a JSON [Further docs](./debugging) |
| default_fallbacks | array of strings | List of fallback models to use if a specific model group is misconfigured / bad. [Further docs](./reliability#default-fallbacks) |
| request_timeout | integer | The timeout for requests in seconds. If not set, the default value is `6000 seconds`. [For reference OpenAI Python SDK defaults to `600 seconds`.](https://github.com/openai/openai-python/blob/main/src/openai/_constants.py) |
| force_ipv4 | boolean | If true, litellm will force ipv4 for all LLM requests. Some users have seen httpx ConnectionError when using ipv6 + Anthropic API |
| content_policy_fallbacks | array of objects | Fallbacks to use when a ContentPolicyViolationError is encountered. [Further docs](./reliability#content-policy-fallbacks) |
| context_window_fallbacks | array of objects | Fallbacks to use when a ContextWindowExceededError is encountered. [Further docs](./reliability#context-window-fallbacks) |
| cache | boolean | If true, enables caching. [Further docs](./caching) |
| cache_params | object | Parameters for the cache. [Further docs](./caching#supported-cache_params-on-proxy-configyaml) |
| disable_end_user_cost_tracking | boolean | If true, turns off end user cost tracking on prometheus metrics + litellm spend logs table on proxy. |
| disable_end_user_cost_tracking_prometheus_only | boolean | If true, turns off end user cost tracking on prometheus metrics only. |
| key_generation_settings | object | Restricts who can generate keys. [Further docs](./virtual_keys.md#restricting-key-generation) |
| disable_add_transform_inline_image_block | boolean | For Fireworks AI models - if true, turns off the auto-add of `#transform=inline` to the url of the image_url, if the model is not a vision model. |
| disable_hf_tokenizer_download | boolean | If true, it defaults to using the openai tokenizer for all models (including huggingface models). |
| enable_json_schema_validation | boolean | If true, enables json schema validation for all requests. |
| disable_copilot_system_to_assistant | boolean | If false (default), converts all 'system' role messages to 'assistant' for GitHub Copilot compatibility. Set to true to disable this behavior. Useful for tools (like Claude Code) that send system messages, which Copilot does not support. |

### general_settings - Reference

| Name | Type | Description |
|------|------|-------------|
| completion_model | string | The default model to use for completions when `model` is not specified in the request |
| disable_spend_logs | boolean | If true, turns off writing each transaction to the database |
| disable_spend_updates | boolean | If true, turns off all spend updates to the DB. Including key/user/team spend updates. |
| disable_master_key_return | boolean | If true, turns off returning master key on UI. (checked on '/user/info' endpoint) |
| disable_retry_on_max_parallel_request_limit_error | boolean | If true, turns off retries when max parallel request limit is reached |
| disable_reset_budget | boolean | If true, turns off reset budget scheduled task |
| disable_adding_master_key_hash_to_db | boolean | If true, turns off storing master key hash in db |
| enable_jwt_auth | boolean | allow proxy admin to auth in via jwt tokens with 'litellm_proxy_admin' in claims. [Doc on JWT Tokens](token_auth) |
| enforce_user_param | boolean | If true, requires all OpenAI endpoint requests to have a 'user' param. [Doc on call hooks](call_hooks)|
| allowed_routes | array of strings | List of allowed proxy API routes a user can access [Doc on controlling allowed routes](enterprise#control-available-public-private-routes)|
| key_management_system | string | Specifies the key management system. [Doc Secret Managers](../secret) |
| master_key | string | The master key for the proxy [Set up Virtual Keys](virtual_keys) |
| database_url | string | The URL for the database connection [Set up Virtual Keys](virtual_keys) |
| database_connection_pool_limit | integer | The limit for database connection pool [Setting DB Connection Pool limit](#configure-db-pool-limits--connection-timeouts) |
| database_connection_timeout | integer | The timeout for database connections in seconds [Setting DB Connection Pool limit, timeout](#configure-db-pool-limits--connection-timeouts) |
| allow_requests_on_db_unavailable | boolean | If true, allows requests to succeed even if DB is unreachable. **Only use this if running LiteLLM in your VPC** This will allow requests to work even when LiteLLM cannot connect to the DB to verify a Virtual Key [Doc on graceful db unavailability](prod#5-if-running-litellm-on-vpc-gracefully-handle-db-unavailability) |
| custom_auth | string | Write your own custom authentication logic [Doc Custom Auth](virtual_keys#custom-auth) |
| max_parallel_requests | integer | The max parallel requests allowed per deployment |
| global_max_parallel_requests | integer | The max parallel requests allowed on the proxy overall |
| infer_model_from_keys | boolean | If true, infers the model from the provided keys |
| background_health_checks | boolean | If true, enables background health checks. [Doc on health checks](health) |
| health_check_interval | integer | The interval for health checks in seconds [Doc on health checks](health) |
| alerting | array of strings | List of alerting methods [Doc on Slack Alerting](alerting) |
| alerting_threshold | integer | The threshold for triggering alerts [Doc on Slack Alerting](alerting) |
| use_client_credentials_pass_through_routes | boolean | If true, uses client credentials for all pass-through routes. [Doc on pass through routes](pass_through) |
| health_check_details | boolean | If false, hides health check details (e.g. remaining rate limit). [Doc on health checks](health) |
| public_routes | List[str] | (Enterprise Feature) Control list of public routes |
| alert_types | List[str] | Control list of alert types to send to slack (Doc on alert types)[./alerting.md] |
| enforced_params | List[str] | (Enterprise Feature) List of params that must be included in all requests to the proxy |
| enable_oauth2_auth | boolean | (Enterprise Feature) If true, enables oauth2.0 authentication |
| use_x_forwarded_for | str | If true, uses the X-Forwarded-For header to get the client IP address |
| service_account_settings | List[Dict[str, Any]] | Set `service_account_settings` if you want to create settings that only apply to service account keys (Doc on service accounts)[./service_accounts.md] | 
| image_generation_model | str | The default model to use for image generation - ignores model set in request |
| store_model_in_db | boolean | If true, enables storing model + credential information in the DB. |
| store_prompts_in_spend_logs | boolean | If true, allows prompts and responses to be stored in the spend logs table. |
| max_request_size_mb | int | The maximum size for requests in MB. Requests above this size will be rejected. |
| max_response_size_mb | int | The maximum size for responses in MB. LLM Responses above this size will not be sent. |
| proxy_budget_rescheduler_min_time | int | The minimum time (in seconds) to wait before checking db for budget resets. **Default is 597 seconds** |
| proxy_budget_rescheduler_max_time | int | The maximum time (in seconds) to wait before checking db for budget resets. **Default is 605 seconds** |
| proxy_batch_write_at | int | Time (in seconds) to wait before batch writing spend logs to the db. **Default is 10 seconds** |
| proxy_batch_polling_interval | int | Time (in seconds) to wait before polling a batch, to check if it's completed. **Default is 6000 seconds (1 hour)** |
| alerting_args | dict | Args for Slack Alerting [Doc on Slack Alerting](./alerting.md) |
| custom_key_generate | str | Custom function for key generation [Doc on custom key generation](./virtual_keys.md#custom--key-generate) |
| allowed_ips | List[str] | List of IPs allowed to access the proxy. If not set, all IPs are allowed. |
| embedding_model | str | The default model to use for embeddings - ignores model set in request |
| default_team_disabled | boolean | If true, users cannot create 'personal' keys (keys with no team_id). |
| alert_to_webhook_url | Dict[str] | [Specify a webhook url for each alert type.](./alerting.md#set-specific-slack-channels-per-alert-type) |
| key_management_settings | List[Dict[str, Any]] | Settings for key management system (e.g. AWS KMS, Azure Key Vault) [Doc on key management](../secret.md) |
| allow_user_auth | boolean | (Deprecated) old approach for user authentication. |
| user_api_key_cache_ttl | int | The time (in seconds) to cache user api keys in memory. |
| disable_prisma_schema_update | boolean | If true, turns off automatic schema updates to DB |
| litellm_key_header_name | str | If set, allows passing LiteLLM keys as a custom header. [Doc on custom headers](./virtual_keys.md#custom-headers) |
| moderation_model | str | The default model to use for moderation. |
| custom_sso | str | Path to a python file that implements custom SSO logic. [Doc on custom SSO](./custom_sso.md) |
| allow_client_side_credentials | boolean | If true, allows passing client side credentials to the proxy. (Useful when testing finetuning models) [Doc on client side credentials](./virtual_keys.md#client-side-credentials) |
| admin_only_routes | List[str] | (Enterprise Feature) List of routes that are only accessible to admin users. [Doc on admin only routes](./enterprise#control-available-public-private-routes) |
| use_azure_key_vault | boolean | If true, load keys from azure key vault | 
| use_google_kms | boolean | If true, load keys from google kms |
| spend_report_frequency | str | Specify how often you want a Spend Report to be sent (e.g. "1d", "2d", "30d") [More on this](./alerting.md#spend-report-frequency) |
| ui_access_mode | Literal["admin_only"] | If set, restricts access to the UI to admin users only. [Docs](./ui.md#restrict-ui-access) |
| litellm_jwtauth | Dict[str, Any] | Settings for JWT authentication. [Docs](./token_auth.md) |
| litellm_license | str | The license key for the proxy. [Docs](../enterprise.md#how-does-deployment-with-enterprise-license-work) |
| oauth2_config_mappings | Dict[str, str] | Define the OAuth2 config mappings | 
| pass_through_endpoints | List[Dict[str, Any]] | Define the pass through endpoints. [Docs](./pass_through) |
| enable_oauth2_proxy_auth | boolean | (Enterprise Feature) If true, enables oauth2.0 authentication |
| forward_openai_org_id | boolean | If true, forwards the OpenAI Organization ID to the backend LLM call (if it's OpenAI). |
| forward_client_headers_to_llm_api | boolean | If true, forwards the client headers (any `x-` headers and `anthropic-beta` headers) to the backend LLM call |
| maximum_spend_logs_retention_period               | str                   | Used to set the max retention time for spend logs in the db, after which they will be auto-purged                                                                                                                                                                                                                             |
| maximum_spend_logs_retention_interval | str | Used to set the interval in which the spend log cleanup task should run in.                                                                                                                                                                                                                                                   |
### router_settings - Reference

:::info

Most values can also be set via `litellm_settings`. If you see overlapping values, settings on `router_settings` will override those on `litellm_settings`.
:::

```yaml
router_settings:
  routing_strategy: usage-based-routing-v2 # Literal["simple-shuffle", "least-busy", "usage-based-routing","latency-based-routing"], default="simple-shuffle"
  redis_host: <your-redis-host>           # string
  redis_password: <your-redis-password>   # string
  redis_port: <your-redis-port>           # string
  enable_pre_call_checks: true            # bool - Before call is made check if a call is within model context window 
  allowed_fails: 3 # cooldown model if it fails > 1 call in a minute. 
  cooldown_time: 30 # (in seconds) how long to cooldown model if fails/min > allowed_fails
  disable_cooldowns: True                  # bool - Disable cooldowns for all models 
  enable_tag_filtering: True                # bool - Use tag based routing for requests
  retry_policy: {                          # Dict[str, int]: retry policy for different types of exceptions
    "AuthenticationErrorRetries": 3,
    "TimeoutErrorRetries": 3,
    "RateLimitErrorRetries": 3,
    "ContentPolicyViolationErrorRetries": 4,
    "InternalServerErrorRetries": 4
  }
  allowed_fails_policy: {
    "BadRequestErrorAllowedFails": 1000, # Allow 1000 BadRequestErrors before cooling down a deployment
    "AuthenticationErrorAllowedFails": 10, # int 
    "TimeoutErrorAllowedFails": 12, # int 
    "RateLimitErrorAllowedFails": 10000, # int 
    "ContentPolicyViolationErrorAllowedFails": 15, # int 
    "InternalServerErrorAllowedFails": 20, # int 
  }
  content_policy_fallbacks=[{"claude-2": ["my-fallback-model"]}] # List[Dict[str, List[str]]]: Fallback model for content policy violations
  fallbacks=[{"claude-2": ["my-fallback-model"]}] # List[Dict[str, List[str]]]: Fallback model for all errors
```

| Name | Type | Description |
|------|------|-------------|
| routing_strategy | string | The strategy used for routing requests. Options: "simple-shuffle", "least-busy", "usage-based-routing", "latency-based-routing". Default is "simple-shuffle". [More information here](../routing) |
| redis_host | string | The host address for the Redis server. **Only set this if you have multiple instances of LiteLLM Proxy and want current tpm/rpm tracking to be shared across them** |
| redis_password | string | The password for the Redis server. **Only set this if you have multiple instances of LiteLLM Proxy and want current tpm/rpm tracking to be shared across them** |
| redis_port | string | The port number for the Redis server. **Only set this if you have multiple instances of LiteLLM Proxy and want current tpm/rpm tracking to be shared across them**|
| enable_pre_call_check | boolean | If true, checks if a call is within the model's context window before making the call. [More information here](reliability) |
| content_policy_fallbacks | array of objects | Specifies fallback models for content policy violations. [More information here](reliability) |
| fallbacks | array of objects | Specifies fallback models for all types of errors. [More information here](reliability) |
| enable_tag_filtering | boolean | If true, uses tag based routing for requests [Tag Based Routing](tag_routing) |
| cooldown_time | integer | The duration (in seconds) to cooldown a model if it exceeds the allowed failures. |
| disable_cooldowns | boolean | If true, disables cooldowns for all models. [More information here](reliability) |
| retry_policy | object | Specifies the number of retries for different types of exceptions. [More information here](reliability) |
| allowed_fails | integer | The number of failures allowed before cooling down a model. [More information here](reliability) |
| allowed_fails_policy | object | Specifies the number of allowed failures for different error types before cooling down a deployment. [More information here](reliability) |
| default_max_parallel_requests | Optional[int] | The default maximum number of parallel requests for a deployment. |
| default_priority | (Optional[int]) | The default priority for a request. Only for '.scheduler_acompletion()'. Default is None. | 
| polling_interval | (Optional[float]) | frequency of polling queue. Only for '.scheduler_acompletion()'. Default is 3ms. |
| max_fallbacks | Optional[int] | The maximum number of fallbacks to try before exiting the call. Defaults to 5. |
| default_litellm_params | Optional[dict] | The default litellm parameters to add to all requests (e.g. `temperature`, `max_tokens`). |
| timeout | Optional[float] | The default timeout for a request. Default is 10 minutes. |
| stream_timeout | Optional[float] | The default timeout for a streaming request. If not set, the 'timeout' value is used. |
| debug_level | Literal["DEBUG", "INFO"] | The debug level for the logging library in the router. Defaults to "INFO". |
| client_ttl | int | Time-to-live for cached clients in seconds. Defaults to 3600. |
| cache_kwargs | dict | Additional keyword arguments for the cache initialization. |
| routing_strategy_args | dict | Additional keyword arguments for the routing strategy - e.g. lowest latency routing default ttl |
| model_group_alias | dict | Model group alias mapping. E.g. `{"claude-3-haiku": "claude-3-haiku-20240229"}` |
| num_retries | int | Number of retries for a request. Defaults to 3. |
| default_fallbacks | Optional[List[str]] | Fallbacks to try if no model group-specific fallbacks are defined. |
| caching_groups | Optional[List[tuple]] | List of model groups for caching across model groups. Defaults to None. - e.g. caching_groups=[("openai-gpt-3.5-turbo", "azure-gpt-3.5-turbo")]|
| alerting_config | AlertingConfig | [SDK-only arg] Slack alerting configuration. Defaults to None. [Further Docs](../routing.md#alerting-) |
| assistants_config | AssistantsConfig | Set on proxy via `assistant_settings`. [Further docs](../assistants.md) |
| set_verbose | boolean | [DEPRECATED PARAM - see debug docs](./debugging.md) If true, sets the logging level to verbose. |
| retry_after | int | Time to wait before retrying a request in seconds. Defaults to 0. If `x-retry-after` is received from LLM API, this value is overridden. |
| provider_budget_config | ProviderBudgetConfig | Provider budget configuration. Use this to set llm_provider budget limits. example $100/day to OpenAI, $100/day to Azure, etc. Defaults to None. [Further Docs](./provider_budget_routing.md) |
| enable_pre_call_checks | boolean | If true, checks if a call is within the model's context window before making the call. [More information here](reliability) |
| model_group_retry_policy | Dict[str, RetryPolicy] | [SDK-only arg] Set retry policy for model groups. |
| context_window_fallbacks | List[Dict[str, List[str]]] | Fallback models for context window violations. |
| redis_url | str | URL for Redis server. **Known performance issue with Redis URL.** |
| cache_responses | boolean | Flag to enable caching LLM Responses, if cache set under `router_settings`. If true, caches responses. Defaults to False. |
| router_general_settings | RouterGeneralSettings | [SDK-Only] Router general settings - contains optimizations like 'async_only_mode'. [Docs](../routing.md#router-general-settings) |
| optional_pre_call_checks | List[str] | List of pre-call checks to add to the router. Currently supported: 'router_budget_limiting', 'prompt_caching' |
| ignore_invalid_deployments | boolean | If true, ignores invalid deployments. Default for proxy is True - to prevent invalid models from blocking other models from being loaded. |


### environment variables - Reference

| Name | Description |
|------|-------------|
| ACTIONS_ID_TOKEN_REQUEST_TOKEN | Token for requesting ID in GitHub Actions
| ACTIONS_ID_TOKEN_REQUEST_URL | URL for requesting ID token in GitHub Actions
| AGENTOPS_ENVIRONMENT | Environment for AgentOps logging integration
| AGENTOPS_API_KEY | API Key for AgentOps logging integration
| AGENTOPS_SERVICE_NAME | Service Name for AgentOps logging integration
| AISPEND_ACCOUNT_ID | Account ID for AI Spend
| AISPEND_API_KEY | API Key for AI Spend
| AIOHTTP_TRUST_ENV | Flag to enable aiohttp trust environment. When this is set to True, aiohttp will respect HTTP(S)_PROXY env vars. **Default is False**
| ALLOWED_EMAIL_DOMAINS | List of email domains allowed for access
| ARIZE_API_KEY | API key for Arize platform integration
| ARIZE_SPACE_KEY | Space key for Arize platform
| ARGILLA_BATCH_SIZE | Batch size for Argilla logging
| ARGILLA_API_KEY | API key for Argilla platform
| ARGILLA_SAMPLING_RATE | Sampling rate for Argilla logging
| ARGILLA_DATASET_NAME | Dataset name for Argilla logging
| ARGILLA_BASE_URL | Base URL for Argilla service
| ATHINA_API_KEY | API key for Athina service
| ATHINA_BASE_URL | Base URL for Athina service (defaults to `https://log.athina.ai`)
| AUTH_STRATEGY | Strategy used for authentication (e.g., OAuth, API key)
| ANTHROPIC_API_KEY | API key for Anthropic service
| AWS_ACCESS_KEY_ID | Access Key ID for AWS services
| AWS_PROFILE_NAME | AWS CLI profile name to be used
| AWS_REGION_NAME | Default AWS region for service interactions
| AWS_ROLE_NAME | Role name for AWS IAM usage
| AWS_SECRET_ACCESS_KEY | Secret Access Key for AWS services
| AWS_SESSION_NAME | Name for AWS session
| AWS_WEB_IDENTITY_TOKEN | Web identity token for AWS
| AZURE_API_VERSION | Version of the Azure API being used
| AZURE_AUTHORITY_HOST | Azure authority host URL
| AZURE_CLIENT_ID | Client ID for Azure services
| AZURE_CLIENT_SECRET | Client secret for Azure services
| AZURE_CODE_INTERPRETER_COST_PER_SESSION | Cost per session for Azure Code Interpreter service
| AZURE_COMPUTER_USE_INPUT_COST_PER_1K_TOKENS | Input cost per 1K tokens for Azure Computer Use service
| AZURE_COMPUTER_USE_OUTPUT_COST_PER_1K_TOKENS | Output cost per 1K tokens for Azure Computer Use service
| AZURE_TENANT_ID | Tenant ID for Azure Active Directory
| AZURE_USERNAME | Username for Azure services, use in conjunction with AZURE_PASSWORD for azure ad token with basic username/password workflow
| AZURE_PASSWORD | Password for Azure services, use in conjunction with AZURE_USERNAME for azure ad token with basic username/password workflow
| AZURE_FEDERATED_TOKEN_FILE | File path to Azure federated token
| AZURE_FILE_SEARCH_COST_PER_GB_PER_DAY | Cost per GB per day for Azure File Search service
| AZURE_SCOPE | For EntraID Auth, Scope for Azure services, defaults to "https://cognitiveservices.azure.com/.default"
| AZURE_KEY_VAULT_URI | URI for Azure Key Vault
| AZURE_OPERATION_POLLING_TIMEOUT | Timeout in seconds for Azure operation polling
| AZURE_STORAGE_ACCOUNT_KEY | The Azure Storage Account Key to use for Authentication to Azure Blob Storage logging
| AZURE_STORAGE_ACCOUNT_NAME | Name of the Azure Storage Account to use for logging to Azure Blob Storage
| AZURE_STORAGE_FILE_SYSTEM | Name of the Azure Storage File System to use for logging to Azure Blob Storage.  (Typically the Container name)
| AZURE_STORAGE_TENANT_ID | The Application Tenant ID to use for Authentication to Azure Blob Storage logging
| AZURE_STORAGE_CLIENT_ID | The Application Client ID to use for Authentication to Azure Blob Storage logging
| AZURE_STORAGE_CLIENT_SECRET | The Application Client Secret to use for Authentication to Azure Blob Storage logging
| AZURE_VECTOR_STORE_COST_PER_GB_PER_DAY | Cost per GB per day for Azure Vector Store service
| BATCH_STATUS_POLL_INTERVAL_SECONDS | Interval in seconds for polling batch status. Default is 3600 (1 hour)
| BATCH_STATUS_POLL_MAX_ATTEMPTS | Maximum number of attempts for polling batch status. Default is 24 (for 24 hours)
| BEDROCK_MAX_POLICY_SIZE | Maximum size for Bedrock policy. Default is 75
| BERRISPEND_ACCOUNT_ID | Account ID for BerriSpend service
| BRAINTRUST_API_KEY | API key for Braintrust integration
| CACHED_STREAMING_CHUNK_DELAY | Delay in seconds for cached streaming chunks. Default is 0.02
| CIRCLE_OIDC_TOKEN | OpenID Connect token for CircleCI
| CIRCLE_OIDC_TOKEN_V2 | Version 2 of the OpenID Connect token for CircleCI
| CONFIG_FILE_PATH | File path for configuration file
| CONFIDENT_API_KEY | API key for DeepEval integration
| CUSTOM_TIKTOKEN_CACHE_DIR | Custom directory for Tiktoken cache
| CONFIDENT_API_KEY | API key for Confident AI (Deepeval) Logging service
| DATABASE_HOST | Hostname for the database server
| DATABASE_NAME | Name of the database
| DATABASE_PASSWORD | Password for the database user
| DATABASE_PORT | Port number for database connection
| DATABASE_SCHEMA | Schema name used in the database
| DATABASE_URL | Connection URL for the database
| DATABASE_USER | Username for database connection
| DATABASE_USERNAME | Alias for database user
| DATABRICKS_API_BASE | Base URL for Databricks API
| DAYS_IN_A_MONTH | Days in a month for calculation purposes. Default is 28
| DAYS_IN_A_WEEK | Days in a week for calculation purposes. Default is 7
| DAYS_IN_A_YEAR | Days in a year for calculation purposes. Default is 365
| DD_BASE_URL | Base URL for Datadog integration
| DATADOG_BASE_URL | (Alternative to DD_BASE_URL) Base URL for Datadog integration
| _DATADOG_BASE_URL | (Alternative to DD_BASE_URL) Base URL for Datadog integration
| DD_API_KEY | API key for Datadog integration
| DD_SITE | Site URL for Datadog (e.g., datadoghq.com)
| DD_SOURCE | Source identifier for Datadog logs
| DD_TRACER_STREAMING_CHUNK_YIELD_RESOURCE | Resource name for Datadog tracing of streaming chunk yields. Default is "streaming.chunk.yield"
| DD_ENV | Environment identifier for Datadog logs. Only supported for `datadog_llm_observability` callback
| DD_SERVICE | Service identifier for Datadog logs. Defaults to "litellm-server"
| DD_VERSION | Version identifier for Datadog logs. Defaults to "unknown"
| DEBUG_OTEL | Enable debug mode for OpenTelemetry
| DEFAULT_ALLOWED_FAILS | Maximum failures allowed before cooling down a model. Default is 3
| DEFAULT_ANTHROPIC_CHAT_MAX_TOKENS | Default maximum tokens for Anthropic chat completions. Default is 4096
| DEFAULT_BATCH_SIZE | Default batch size for operations. Default is 512
| DEFAULT_COOLDOWN_TIME_SECONDS | Duration in seconds to cooldown a model after failures. Default is 5
| DEFAULT_CRON_JOB_LOCK_TTL_SECONDS | Time-to-live for cron job locks in seconds. Default is 60 (1 minute)
| DEFAULT_FAILURE_THRESHOLD_PERCENT | Threshold percentage of failures to cool down a deployment. Default is 0.5 (50%)
| DEFAULT_FLUSH_INTERVAL_SECONDS | Default interval in seconds for flushing operations. Default is 5
| DEFAULT_HEALTH_CHECK_INTERVAL | Default interval in seconds for health checks. Default is 300 (5 minutes)
| DEFAULT_IMAGE_HEIGHT | Default height for images. Default is 300
| DEFAULT_IMAGE_TOKEN_COUNT | Default token count for images. Default is 250
| DEFAULT_IMAGE_WIDTH | Default width for images. Default is 300
| DEFAULT_IN_MEMORY_TTL | Default time-to-live for in-memory cache in seconds. Default is 5
| DEFAULT_MANAGEMENT_OBJECT_IN_MEMORY_CACHE_TTL | Default time-to-live in seconds for management objects (User, Team, Key, Organization) in memory cache. Default is 60 seconds.
| DEFAULT_MAX_LRU_CACHE_SIZE | Default maximum size for LRU cache. Default is 16
| DEFAULT_MAX_RECURSE_DEPTH | Default maximum recursion depth. Default is 100
| DEFAULT_MAX_RECURSE_DEPTH_SENSITIVE_DATA_MASKER | Default maximum recursion depth for sensitive data masker. Default is 10
| DEFAULT_MAX_RETRIES | Default maximum retry attempts. Default is 2
| DEFAULT_MAX_TOKENS | Default maximum tokens for LLM calls. Default is 4096
| DEFAULT_MAX_TOKENS_FOR_TRITON | Default maximum tokens for Triton models. Default is 2000
| DEFAULT_MOCK_RESPONSE_COMPLETION_TOKEN_COUNT | Default token count for mock response completions. Default is 20
| DEFAULT_MOCK_RESPONSE_PROMPT_TOKEN_COUNT | Default token count for mock response prompts. Default is 10
| DEFAULT_MODEL_CREATED_AT_TIME | Default creation timestamp for models. Default is 1677610602
| DEFAULT_PROMPT_INJECTION_SIMILARITY_THRESHOLD | Default threshold for prompt injection similarity. Default is 0.7
| DEFAULT_POLLING_INTERVAL | Default polling interval for schedulers in seconds. Default is 0.03
| DEFAULT_REASONING_EFFORT_DISABLE_THINKING_BUDGET | Default reasoning effort disable thinking budget. Default is 0
| DEFAULT_REASONING_EFFORT_HIGH_THINKING_BUDGET | Default high reasoning effort thinking budget. Default is 4096
| DEFAULT_REASONING_EFFORT_LOW_THINKING_BUDGET | Default low reasoning effort thinking budget. Default is 1024
| DEFAULT_REASONING_EFFORT_MEDIUM_THINKING_BUDGET | Default medium reasoning effort thinking budget. Default is 2048
| DEFAULT_REDIS_SYNC_INTERVAL | Default Redis synchronization interval in seconds. Default is 1
| DEFAULT_REPLICATE_GPU_PRICE_PER_SECOND | Default price per second for Replicate GPU. Default is 0.001400
| DEFAULT_REPLICATE_POLLING_DELAY_SECONDS | Default delay in seconds for Replicate polling. Default is 1
| DEFAULT_REPLICATE_POLLING_RETRIES | Default number of retries for Replicate polling. Default is 5
| DEFAULT_SQS_BATCH_SIZE | Default batch size for SQS logging. Default is 512
| DEFAULT_SQS_FLUSH_INTERVAL_SECONDS | Default flush interval for SQS logging. Default is 10
| DEFAULT_S3_BATCH_SIZE | Default batch size for S3 logging. Default is 512
| DEFAULT_S3_FLUSH_INTERVAL_SECONDS | Default flush interval for S3 logging. Default is 10
| DEFAULT_SLACK_ALERTING_THRESHOLD | Default threshold for Slack alerting. Default is 300
| DEFAULT_SOFT_BUDGET | Default soft budget for LiteLLM proxy keys. Default is 50.0
| DEFAULT_TRIM_RATIO | Default ratio of tokens to trim from prompt end. Default is 0.75
| DIRECT_URL | Direct URL for service endpoint
| DISABLE_ADMIN_UI | Toggle to disable the admin UI
| DISABLE_AIOHTTP_TRANSPORT | Flag to disable aiohttp transport. When this is set to True, litellm will use httpx instead of aiohttp. **Default is False**
| DISABLE_AIOHTTP_TRUST_ENV | Flag to disable aiohttp trust environment. When this is set to True, litellm will not trust the environment for aiohttp eg. `HTTP_PROXY` and `HTTPS_PROXY` environment variables will not be used when this is set to True. **Default is False**
| DISABLE_SCHEMA_UPDATE | Toggle to disable schema updates
| DOCS_DESCRIPTION | Description text for documentation pages
| DOCS_FILTERED | Flag indicating filtered documentation
| DOCS_TITLE | Title of the documentation pages
| DOCS_URL | The path to the Swagger API documentation. **By default this is "/"**
| EMAIL_LOGO_URL | URL for the logo used in emails
| EMAIL_SUPPORT_CONTACT | Support contact email address
| EMAIL_SIGNATURE | Custom HTML footer/signature for all emails. Can include HTML tags for formatting and links.
| EMAIL_SUBJECT_INVITATION | Custom subject template for invitation emails. 
| EMAIL_SUBJECT_KEY_CREATED | Custom subject template for key creation emails. 
| EXPERIMENTAL_MULTI_INSTANCE_RATE_LIMITING | Flag to enable new multi-instance rate limiting. **Default is False**
| FIREWORKS_AI_4_B | Size parameter for Fireworks AI 4B model. Default is 4
| FIREWORKS_AI_16_B | Size parameter for Fireworks AI 16B model. Default is 16
| FIREWORKS_AI_56_B_MOE | Size parameter for Fireworks AI 56B MOE model. Default is 56
| FIREWORKS_AI_80_B | Size parameter for Fireworks AI 80B model. Default is 80
| FIREWORKS_AI_176_B_MOE | Size parameter for Fireworks AI 176B MOE model. Default is 176
| FUNCTION_DEFINITION_TOKEN_COUNT | Token count for function definitions. Default is 9
| GALILEO_BASE_URL | Base URL for Galileo platform
| GALILEO_PASSWORD | Password for Galileo authentication
| GALILEO_PROJECT_ID | Project ID for Galileo usage
| GALILEO_USERNAME | Username for Galileo authentication
| GOOGLE_SECRET_MANAGER_PROJECT_ID | Project ID for Google Secret Manager
| GCS_BUCKET_NAME | Name of the Google Cloud Storage bucket
| GCS_PATH_SERVICE_ACCOUNT | Path to the Google Cloud service account JSON file
| GCS_FLUSH_INTERVAL | Flush interval for GCS logging (in seconds). Specify how often you want a log to be sent to GCS. **Default is 20 seconds**
| GCS_BATCH_SIZE | Batch size for GCS logging. Specify after how many logs you want to flush to GCS. If `BATCH_SIZE` is set to 10, logs are flushed every 10 logs. **Default is 2048**
| GCS_PUBSUB_TOPIC_ID | PubSub Topic ID to send LiteLLM SpendLogs to.
| GCS_PUBSUB_PROJECT_ID | PubSub Project ID to send LiteLLM SpendLogs to.
| GENERIC_AUTHORIZATION_ENDPOINT | Authorization endpoint for generic OAuth providers
| GENERIC_CLIENT_ID | Client ID for generic OAuth providers
| GENERIC_CLIENT_SECRET | Client secret for generic OAuth providers
| GENERIC_CLIENT_STATE | State parameter for generic client authentication
| GENERIC_SSO_HEADERS | Comma-separated list of additional headers to add to the request - e.g. Authorization=Bearer `<token>`, Content-Type=application/json, etc.
| GENERIC_INCLUDE_CLIENT_ID | Include client ID in requests for OAuth
| GENERIC_SCOPE | Scope settings for generic OAuth providers
| GENERIC_TOKEN_ENDPOINT | Token endpoint for generic OAuth providers
| GENERIC_USER_DISPLAY_NAME_ATTRIBUTE | Attribute for user's display name in generic auth
| GENERIC_USER_EMAIL_ATTRIBUTE | Attribute for user's email in generic auth
| GENERIC_USER_FIRST_NAME_ATTRIBUTE | Attribute for user's first name in generic auth
| GENERIC_USER_ID_ATTRIBUTE | Attribute for user ID in generic auth
| GENERIC_USER_LAST_NAME_ATTRIBUTE | Attribute for user's last name in generic auth
| GENERIC_USER_PROVIDER_ATTRIBUTE | Attribute specifying the user's provider
| GENERIC_USER_ROLE_ATTRIBUTE | Attribute specifying the user's role
| GENERIC_USERINFO_ENDPOINT | Endpoint to fetch user information in generic OAuth
| GALILEO_BASE_URL | Base URL for Galileo platform
| GALILEO_PASSWORD | Password for Galileo authentication
| GALILEO_PROJECT_ID | Project ID for Galileo usage
| GALILEO_USERNAME | Username for Galileo authentication
| GITHUB_COPILOT_TOKEN_DIR | Directory to store GitHub Copilot token for `github_copilot` llm provider
| GITHUB_COPILOT_API_KEY_FILE | File to store GitHub Copilot API key for `github_copilot` llm provider
| GITHUB_COPILOT_ACCESS_TOKEN_FILE | File to store GitHub Copilot access token for `github_copilot` llm provider
| GREENSCALE_API_KEY | API key for Greenscale service
| GREENSCALE_ENDPOINT | Endpoint URL for Greenscale service
| GOOGLE_APPLICATION_CREDENTIALS | Path to Google Cloud credentials JSON file
| GOOGLE_CLIENT_ID | Client ID for Google OAuth
| GOOGLE_CLIENT_SECRET | Client secret for Google OAuth
| GOOGLE_KMS_RESOURCE_NAME | Name of the resource in Google KMS
| GUARDRAILS_AI_API_BASE | Base URL for Guardrails AI API
| HEALTH_CHECK_TIMEOUT_SECONDS | Timeout in seconds for health checks. Default is 60
| HF_API_BASE | Base URL for Hugging Face API
| HCP_VAULT_ADDR | Address for [Hashicorp Vault Secret Manager](../secret.md#hashicorp-vault)
| HCP_VAULT_CLIENT_CERT | Path to client certificate for [Hashicorp Vault Secret Manager](../secret.md#hashicorp-vault)
| HCP_VAULT_CLIENT_KEY | Path to client key for [Hashicorp Vault Secret Manager](../secret.md#hashicorp-vault)
| HCP_VAULT_NAMESPACE | Namespace for [Hashicorp Vault Secret Manager](../secret.md#hashicorp-vault)
| HCP_VAULT_TOKEN | Token for [Hashicorp Vault Secret Manager](../secret.md#hashicorp-vault)
| HCP_VAULT_CERT_ROLE | Role for [Hashicorp Vault Secret Manager Auth](../secret.md#hashicorp-vault)
| HELICONE_API_KEY | API key for Helicone service
| HELICONE_API_BASE | Base URL for Helicone service, defaults to `https://api.helicone.ai`
| HOSTNAME | Hostname for the server, this will be [emitted to `datadog` logs](https://docs.litellm.ai/docs/proxy/logging#datadog)
| HOURS_IN_A_DAY | Hours in a day for calculation purposes. Default is 24
| HUGGINGFACE_API_BASE | Base URL for Hugging Face API
| HUGGINGFACE_API_KEY | API key for Hugging Face API
| HUMANLOOP_PROMPT_CACHE_TTL_SECONDS | Time-to-live in seconds for cached prompts in Humanloop. Default is 60
| IAM_TOKEN_DB_AUTH | IAM token for database authentication
| INITIAL_RETRY_DELAY | Initial delay in seconds for retrying requests. Default is 0.5
| JITTER | Jitter factor for retry delay calculations. Default is 0.75
| JSON_LOGS | Enable JSON formatted logging
| JWT_AUDIENCE | Expected audience for JWT tokens
| JWT_PUBLIC_KEY_URL | URL to fetch public key for JWT verification
| LAGO_API_BASE | Base URL for Lago API
| LAGO_API_CHARGE_BY | Parameter to determine charge basis in Lago
| LAGO_API_EVENT_CODE | Event code for Lago API events
| LAGO_API_KEY | API key for accessing Lago services
| LANGFUSE_DEBUG | Toggle debug mode for Langfuse
| LANGFUSE_FLUSH_INTERVAL | Interval for flushing Langfuse logs
| LANGFUSE_TRACING_ENVIRONMENT | Environment for Langfuse tracing
| LANGFUSE_HOST | Host URL for Langfuse service
| LANGFUSE_PUBLIC_KEY | Public key for Langfuse authentication
| LANGFUSE_RELEASE | Release version of Langfuse integration
| LANGFUSE_SECRET_KEY | Secret key for Langfuse authentication
| LANGSMITH_API_KEY | API key for Langsmith platform
| LANGSMITH_BASE_URL | Base URL for Langsmith service
| LANGSMITH_BATCH_SIZE | Batch size for operations in Langsmith
| LANGSMITH_DEFAULT_RUN_NAME | Default name for Langsmith run
| LANGSMITH_PROJECT | Project name for Langsmith integration
| LANGSMITH_SAMPLING_RATE | Sampling rate for Langsmith logging
| LANGTRACE_API_KEY | API key for Langtrace service
| LASSO_API_BASE | Base URL for Lasso API
| LASSO_API_KEY | API key for Lasso service
| LASSO_USER_ID | User ID for Lasso service
| LASSO_CONVERSATION_ID | Conversation ID for Lasso service
| LENGTH_OF_LITELLM_GENERATED_KEY | Length of keys generated by LiteLLM. Default is 16
| LITERAL_API_KEY | API key for Literal integration
| LITERAL_API_URL | API URL for Literal service
| LITERAL_BATCH_SIZE | Batch size for Literal operations
| LITELLM_DONT_SHOW_FEEDBACK_BOX | Flag to hide feedback box in LiteLLM UI
| LITELLM_DROP_PARAMS | Parameters to drop in LiteLLM requests
| LITELLM_MODIFY_PARAMS | Parameters to modify in LiteLLM requests
| LITELLM_EMAIL | Email associated with LiteLLM account
| LITELLM_GLOBAL_MAX_PARALLEL_REQUEST_RETRIES | Maximum retries for parallel requests in LiteLLM
| LITELLM_GLOBAL_MAX_PARALLEL_REQUEST_RETRY_TIMEOUT | Timeout for retries of parallel requests in LiteLLM
| LITELLM_MIGRATION_DIR | Custom migrations directory for prisma migrations, used for baselining db in read-only file systems.
| LITELLM_HOSTED_UI | URL of the hosted UI for LiteLLM
| LITELM_ENVIRONMENT | Environment of LiteLLM Instance, used by logging services. Currently only used by DeepEval.
| LITELLM_LICENSE | License key for LiteLLM usage
| LITELLM_LOCAL_MODEL_COST_MAP | Local configuration for model cost mapping in LiteLLM
| LITELLM_LOG | Enable detailed logging for LiteLLM
| LITELLM_MASTER_KEY | Master key for proxy authentication
| LITELLM_MODE | Operating mode for LiteLLM (e.g., production, development)
| LITELLM_RATE_LIMIT_WINDOW_SIZE | Rate limit window size for LiteLLM. Default is 60
| LITELLM_SALT_KEY | Salt key for encryption in LiteLLM
| LITELLM_SECRET_AWS_KMS_LITELLM_LICENSE | AWS KMS encrypted license for LiteLLM
| LITELLM_TOKEN | Access token for LiteLLM integration
| LITELLM_PRINT_STANDARD_LOGGING_PAYLOAD | If true, prints the standard logging payload to the console - useful for debugging
| LITELM_ENVIRONMENT | Environment for LiteLLM Instance. This is currently only logged to DeepEval to determine the environment for DeepEval integration.
| LOGFIRE_TOKEN | Token for Logfire logging service
| MAX_EXCEPTION_MESSAGE_LENGTH | Maximum length for exception messages. Default is 2000
| MAX_IN_MEMORY_QUEUE_FLUSH_COUNT | Maximum count for in-memory queue flush operations. Default is 1000
| MAX_LONG_SIDE_FOR_IMAGE_HIGH_RES | Maximum length for the long side of high-resolution images. Default is 2000
| MAX_REDIS_BUFFER_DEQUEUE_COUNT | Maximum count for Redis buffer dequeue operations. Default is 100
| MAX_SHORT_SIDE_FOR_IMAGE_HIGH_RES | Maximum length for the short side of high-resolution images. Default is 768
| MAX_SIZE_IN_MEMORY_QUEUE | Maximum size for in-memory queue. Default is 10000
| MAX_SIZE_PER_ITEM_IN_MEMORY_CACHE_IN_KB | Maximum size in KB for each item in memory cache. Default is 512 or 1024
| MAX_SPENDLOG_ROWS_TO_QUERY | Maximum number of spend log rows to query. Default is 1,000,000
| MAX_TEAM_LIST_LIMIT | Maximum number of teams to list. Default is 20
| MAX_TILE_HEIGHT | Maximum height for image tiles. Default is 512
| MAX_TILE_WIDTH | Maximum width for image tiles. Default is 512
| MAX_TOKEN_TRIMMING_ATTEMPTS | Maximum number of attempts to trim a token message. Default is 10
| MAXIMUM_TRACEBACK_LINES_TO_LOG | Maximum number of lines to log in traceback in LiteLLM Logs UI. Default is 100
| MAX_RETRY_DELAY | Maximum delay in seconds for retrying requests. Default is 8.0
| MAX_LANGFUSE_INITIALIZED_CLIENTS | Maximum number of Langfuse clients to initialize on proxy. Default is 20. This is set since langfuse initializes 1 thread everytime a client is initialized. We've had an incident in the past where we reached 100% cpu utilization because Langfuse was initialized several times.
| MIN_NON_ZERO_TEMPERATURE | Minimum non-zero temperature value. Default is 0.0001
| MINIMUM_PROMPT_CACHE_TOKEN_COUNT | Minimum token count for caching a prompt. Default is 1024
| MISTRAL_API_BASE | Base URL for Mistral API
| MISTRAL_API_KEY | API key for Mistral API
| MICROSOFT_CLIENT_ID | Client ID for Microsoft services
| MICROSOFT_CLIENT_SECRET | Client secret for Microsoft services
| MICROSOFT_TENANT | Tenant ID for Microsoft Azure
| MICROSOFT_SERVICE_PRINCIPAL_ID | Service Principal ID for Microsoft Enterprise Application. (This is an advanced feature if you want litellm to auto-assign members to Litellm Teams based on their Microsoft Entra ID Groups)
| NO_DOCS | Flag to disable Swagger UI documentation
| NO_REDOC | Flag to disable Redoc documentation
| NO_PROXY | List of addresses to bypass proxy
| NON_LLM_CONNECTION_TIMEOUT | Timeout in seconds for non-LLM service connections. Default is 15
| OAUTH_TOKEN_INFO_ENDPOINT | Endpoint for OAuth token info retrieval
| OPENAI_BASE_URL | Base URL for OpenAI API
| OPENAI_API_BASE | Base URL for OpenAI API
| OPENAI_API_KEY | API key for OpenAI services
| OPENAI_FILE_SEARCH_COST_PER_1K_CALLS | Cost per 1000 calls for OpenAI file search. Default is 0.0025
| OPENAI_ORGANIZATION | Organization identifier for OpenAI
| OPENID_BASE_URL | Base URL for OpenID Connect services
| OPENID_CLIENT_ID | Client ID for OpenID Connect authentication
| OPENID_CLIENT_SECRET | Client secret for OpenID Connect authentication
| OPENMETER_API_ENDPOINT | API endpoint for OpenMeter integration
| OPENMETER_API_KEY | API key for OpenMeter services
| OPENMETER_EVENT_TYPE | Type of events sent to OpenMeter
| OTEL_ENDPOINT | OpenTelemetry endpoint for traces
| OTEL_EXPORTER_OTLP_ENDPOINT | OpenTelemetry endpoint for traces
| OTEL_ENVIRONMENT_NAME | Environment name for OpenTelemetry
| OTEL_EXPORTER | Exporter type for OpenTelemetry
| OTEL_EXPORTER_OTLP_PROTOCOL | Exporter type for OpenTelemetry
| OTEL_HEADERS | Headers for OpenTelemetry requests
| OTEL_MODEL_ID | Model ID for OpenTelemetry tracing
| OTEL_EXPORTER_OTLP_HEADERS | Headers for OpenTelemetry requests
| OTEL_SERVICE_NAME | Service name identifier for OpenTelemetry
| OTEL_TRACER_NAME | Tracer name for OpenTelemetry tracing
| PAGERDUTY_API_KEY | API key for PagerDuty Alerting
| PANW_PRISMA_AIRS_API_KEY | API key for PANW Prisma AIRS service
| PANW_PRISMA_AIRS_API_BASE | Base URL for PANW Prisma AIRS service
| PHOENIX_API_KEY | API key for Arize Phoenix
| PHOENIX_COLLECTOR_ENDPOINT | API endpoint for Arize Phoenix
| PHOENIX_COLLECTOR_HTTP_ENDPOINT | API http endpoint for Arize Phoenix
| PILLAR_API_BASE | Base URL for Pillar API Guardrails
| PILLAR_API_KEY | API key for Pillar API Guardrails
| PILLAR_ON_FLAGGED_ACTION | Action to take when content is flagged ('block' or 'monitor')
| POD_NAME | Pod name for the server, this will be [emitted to `datadog` logs](https://docs.litellm.ai/docs/proxy/logging#datadog) as `POD_NAME` 
| PREDIBASE_API_BASE | Base URL for Predibase API
| PRESIDIO_ANALYZER_API_BASE | Base URL for Presidio Analyzer service
| PRESIDIO_ANONYMIZER_API_BASE | Base URL for Presidio Anonymizer service
| PROMETHEUS_BUDGET_METRICS_REFRESH_INTERVAL_MINUTES | Refresh interval in minutes for Prometheus budget metrics. Default is 5
| PROMETHEUS_FALLBACK_STATS_SEND_TIME_HOURS | Fallback time in hours for sending stats to Prometheus. Default is 9
| PROMETHEUS_URL | URL for Prometheus service
| PROMPTLAYER_API_KEY | API key for PromptLayer integration
| PROXY_ADMIN_ID | Admin identifier for proxy server
| PROXY_BASE_URL | Base URL for proxy service
| PROXY_BATCH_WRITE_AT | Time in seconds to wait before batch writing spend logs to the database. Default is 10
| PROXY_BATCH_POLLING_INTERVAL | Time in seconds to wait before polling a batch, to check if it's completed. Default is 6000s (1 hour)
| PROXY_BUDGET_RESCHEDULER_MAX_TIME | Maximum time in seconds to wait before checking database for budget resets. Default is 605
| PROXY_BUDGET_RESCHEDULER_MIN_TIME | Minimum time in seconds to wait before checking database for budget resets. Default is 597
| PROXY_LOGOUT_URL | URL for logging out of the proxy service
| QDRANT_API_BASE | Base URL for Qdrant API
| QDRANT_API_KEY | API key for Qdrant service
| QDRANT_SCALAR_QUANTILE | Scalar quantile for Qdrant operations. Default is 0.99
| QDRANT_URL | Connection URL for Qdrant database
| QDRANT_VECTOR_SIZE | Vector size for Qdrant operations. Default is 1536
| REDIS_CONNECTION_POOL_TIMEOUT | Timeout in seconds for Redis connection pool. Default is 5
| REDIS_HOST | Hostname for Redis server
| REDIS_PASSWORD | Password for Redis service
| REDIS_PORT | Port number for Redis server
| REDIS_SOCKET_TIMEOUT | Timeout in seconds for Redis socket operations. Default is 0.1
| REDOC_URL | The path to the Redoc Fast API documentation. **By default this is "/redoc"**
| REPEATED_STREAMING_CHUNK_LIMIT | Limit for repeated streaming chunks to detect looping. Default is 100
| REPLICATE_MODEL_NAME_WITH_ID_LENGTH | Length of Replicate model names with ID. Default is 64
| REPLICATE_POLLING_DELAY_SECONDS | Delay in seconds for Replicate polling operations. Default is 0.5
| REQUEST_TIMEOUT | Timeout in seconds for requests. Default is 6000
| ROUTER_MAX_FALLBACKS | Maximum number of fallbacks for router. Default is 5
| SECRET_MANAGER_REFRESH_INTERVAL | Refresh interval in seconds for secret manager. Default is 86400 (24 hours)
| SEPARATE_HEALTH_APP | If set to '1', runs health endpoints on a separate ASGI app and port. Default: '0'.
| SEPARATE_HEALTH_PORT | Port for the separate health endpoints app. Only used if SEPARATE_HEALTH_APP=1. Default: 4001.
| SERVER_ROOT_PATH | Root path for the server application
| SET_VERBOSE | Flag to enable verbose logging
| SINGLE_DEPLOYMENT_TRAFFIC_FAILURE_THRESHOLD | Minimum number of requests to consider "reasonable traffic" for single-deployment cooldown logic. Default is 1000
| SLACK_DAILY_REPORT_FREQUENCY | Frequency of daily Slack reports (e.g., daily, weekly)
| SLACK_WEBHOOK_URL | Webhook URL for Slack integration
| SMTP_HOST | Hostname for the SMTP server
| SMTP_PASSWORD | Password for SMTP authentication (do not set if SMTP does not require auth)
| SMTP_PORT | Port number for SMTP server
| SMTP_SENDER_EMAIL | Email address used as the sender in SMTP transactions
| SMTP_SENDER_LOGO | Logo used in emails sent via SMTP
| SMTP_TLS | Flag to enable or disable TLS for SMTP connections
| SMTP_USERNAME | Username for SMTP authentication (do not set if SMTP does not require auth)
| SPEND_LOGS_URL | URL for retrieving spend logs
| SPEND_LOG_CLEANUP_BATCH_SIZE | Number of logs deleted per batch during cleanup. Default is 1000
| SSL_CERTIFICATE | Path to the SSL certificate file
| SSL_SECURITY_LEVEL | [BETA] Security level for SSL/TLS connections. E.g. `DEFAULT@SECLEVEL=1`
| SSL_VERIFY | Flag to enable or disable SSL certificate verification
| SSL_CERT_FILE | Path to the SSL certificate file for custom CA bundle
| SUPABASE_KEY | API key for Supabase service
| SUPABASE_URL | Base URL for Supabase instance
| STORE_MODEL_IN_DB | If true, enables storing model + credential information in the DB. 
| SYSTEM_MESSAGE_TOKEN_COUNT | Token count for system messages. Default is 4
| TEST_EMAIL_ADDRESS | Email address used for testing purposes
| TOGETHER_AI_4_B | Size parameter for Together AI 4B model. Default is 4
| TOGETHER_AI_8_B | Size parameter for Together AI 8B model. Default is 8
| TOGETHER_AI_21_B | Size parameter for Together AI 21B model. Default is 21
| TOGETHER_AI_41_B | Size parameter for Together AI 41B model. Default is 41
| TOGETHER_AI_80_B | Size parameter for Together AI 80B model. Default is 80
| TOGETHER_AI_110_B | Size parameter for Together AI 110B model. Default is 110
| TOGETHER_AI_EMBEDDING_150_M | Size parameter for Together AI 150M embedding model. Default is 150
| TOGETHER_AI_EMBEDDING_350_M | Size parameter for Together AI 350M embedding model. Default is 350
| TOOL_CHOICE_OBJECT_TOKEN_COUNT | Token count for tool choice objects. Default is 4
| UI_LOGO_PATH | Path to the logo image used in the UI
| UI_PASSWORD | Password for accessing the UI
| UI_USERNAME | Username for accessing the UI
| UPSTREAM_LANGFUSE_DEBUG | Flag to enable debugging for upstream Langfuse
| UPSTREAM_LANGFUSE_HOST | Host URL for upstream Langfuse service
| UPSTREAM_LANGFUSE_PUBLIC_KEY | Public key for upstream Langfuse authentication
| UPSTREAM_LANGFUSE_RELEASE | Release version identifier for upstream Langfuse
| UPSTREAM_LANGFUSE_SECRET_KEY | Secret key for upstream Langfuse authentication
| USE_AWS_KMS | Flag to enable AWS Key Management Service for encryption
| USE_PRISMA_MIGRATE | Flag to use prisma migrate instead of prisma db push. Recommended for production environments.
| WEBHOOK_URL | URL for receiving webhooks from external services
| SPEND_LOG_RUN_LOOPS | Constant for setting how many runs of 1000 batch deletes should spend_log_cleanup task run |
| SPEND_LOG_CLEANUP_BATCH_SIZE | Number of logs deleted per batch during cleanup. Default is 1000 |
