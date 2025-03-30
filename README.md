<div align="center">

# AI Agent Models

This repository contains common templates that to be used in chatting and auto
completion tasks.

</div>

## Roadmap

AI Agent Models offers:

- [x] StarCoder
  - [x] Autocompletion
- [x] CodeLlama
  - [x] Autocompletion
- [x] Mistral
  - [x] Chatting
- [x] DeepSeekCoder
  - [x] Autocompletion
- [x] CodeGemma
  - [x] Autocompletion
- [x] CodeQwen
  - [x] Chatting
- [x] Qwen2.5-Coder
  - [x] Autocompletion
- [x] DeepSeekCoder-v2
  - [x] Autocompletion
- [x] Jina Embeddings V2 Code
  - [x] Embedding
- [x] Nomic Embed Text
  - [x] Embedding
- [x] Yi Coder
  - [x] Chatting

## How to use

There is example configuration that I used for config neovim.

1. Deadly simple autocompletion for Neovim. Which is not depended on `cmp`.

```lua
require('bropilot').setup({
  auto_suggest = true,
  excluded_filetypes = {},
  model = "qwen2.5-coder:7b-base",
  model_params = {
    num_ctx = 32768,
    num_ctx = 8192,
    num_predict = -2,
    temperature = 0.2,
    top_p = 0.95,
    stop = { "<|fim_pad|>", "<|endoftext|>" },
  },
  prompt = {
    prefix = "<|fim_prefix|>",
    suffix = "<|fim_suffix|>",
    middle = "<|fim_middle|>",
  },
  debounce = 500,
  keymap = {
    accept_word = "<C-Right>",
    accept_line = "<S-Right>",
    accept_block = "<Tab>",
    suggest = "<C-Down>",
  },
  ollama_url = "http://localhost:11434/api",
})
```

2. Tabby Agent for projects required highly security

```nix
# Tabby Agent with ollama as the server.

# OS base configuration
# For linux, the configuration is in both system level & user level (home manager) to use
# optimization hardware dependencies.
# For Mac, the configuration is setting at user level (home manager) because of
# poorly support from Apple.

tabby = lib.mkOption {
      type = lib.types.submodule {
        options = {
          port = mkIntOption 11001 "Port";
          localRepos = mkListOption (subModuleType {
            name = mkStrOption "" "Name of repository";
            repo = mkStrOption "" "Repositories";
          }) [ ] "Local repositories for indexing | preloading | embedding tasks";
          model = lib.mkOption {
            description = "Tabby Model";
            type = subModuleType {
              chat = mkAttrsOption lib.types.str { } "Chat model";
              completion = mkAttrsOption lib.types.str { } "Completion model";
              embedding = mkAttrsOption lib.types.str { } "Embedding model";
            };
          };
        };
      };
};
# ...
# Configuration tabby models
tabby = let
          ollamaPort = 11000;
        in
        {
          port = ollamaPort;
          repos = [];
          model = {
            chat = {
              kind = "openai/chat";
              model_name = "qwen2.5-coder:7b-instruct";
              api_endpoint = "http://localhost:${builtins.toString ollamaPort}/v1";
            };
            completion = {
              kind = "ollama/completion";
              api_endpoint = "http://localhost:${builtins.toString ollamaPort}";
              model_name = "qwen2.5-coder:7b-base";
              prompt_template = "<|fim_prefix|> {prefix} <|fim_suffix|>{suffix} <|fim_middle|>";
            };
            embedding = {
              kind = "ollama/embedding";
              model_name = "nomic-embed-text:latest";
              api_endpoint = "http://localhost:${builtins.toString ollamaPort}";
            };
          };
        };
# ...

# Home manager configuration
# ...
home.file = let 
        attrsToString =
            attrs:
            let
            keys = builtins.attrNames attrs;
            keyValues = map (k: "${k} = \"${attrs.${k}}\"") keys;
            in
            builtins.concatStringsSep "\n" keyValues;

        mkLocalRepositories =
            repositories:
            builtins.concatStringsSep "\n" (
            map (r: ''
                [[repositories]]
                name = "${r.name}"
                git_url = "file:///${r.repo}"
            '') repositories
            );

        deviceArg =
            if pkgs.stdenv.isDarwin then
            "--device metal"
            else if osConfig.${namespace}.gpu-drivers.nvidia.enable then
            "--device cuda"
            else
            ""; # TODO: support AMD driver. Currently, I don't have any AMD device.
        in
        {
        ".tabby/config.toml".text = ''
          [server]
          endpoint = "http://127.0.0.1:${builtins.toString osCfg.tabby.port}"
          completion_timeout = 15000

          [model.chat.http]
          ${attrsToString osCfg.tabby.model.chat}

          [model.completion.http]
          ${attrsToString osCfg.tabby.model.completion}

          [model.embedding.http]
          ${attrsToString osCfg.tabby.model.embedding}

          ${mkLocalRepositories osCfg.tabby.localRepos}
        '';
        ".tabby-client/agent/config.toml".text = ''
          [server]
          endpoint = "http://localhost:${builtins.toString osCfg.tabby.port}"
          token = "${builtins.getEnv "TABBY_TOKEN"}"

          [logs]
          level = "info"

          [anonymousUsageTracking]
          disable = true
        '';
      };

### Systemd service
# Running tabby agent as a service.
systemd.user.services.tabby = {
      Unit = {
        Description = "User-level Tabby Service";
        After = [ "ollama.service" ];
      };
      Service = {
        Type = "simple";
        Environment = ''
          TABBY_WEBSERVER_JWT_TOKEN_SECRET=${builtins.getEnv "TABBY_WEBSERVER_JWT_TOKEN_SECRET"}
        '';
        ExecStart = ''
          ${pkgs.tabby}/bin/tabby serve \
            --port ${builtins.toString osCfg.tabby.port} \
            ${deviceArg}
        '';
        Restart = "on-failure";
      };
      Install = {
        WantedBy = [ "default.target" ];
      };
    };
```

### Prompt Template

Prompt template represents the template that can be used for auto completion
tasks, for the model that supports FIM (Fill In the Middle) model.

### Chat Template

For chating purpose.

## Tasks

- [x] Research some useful model prompts to be used with `Continue` in VSCode
      and `Bropilot` | `CMP-AI` | `Tabby Agent` | `Avante.nvim` |
      `Codecompanion.nvim` in Neovim.
- [x] Implemented!
- [x] Yay Release!!!
