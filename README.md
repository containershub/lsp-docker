# LSP Docker - Containerize LSPs
Docker setup for running LSPs in a container for Neovim.

> [!WARNING]
> There might be better ways to setup your LSPs with Docker but this is my
> attempt and I'm totally fine with it. If you want to use this use it with
> caution as it might not be very convenient because it takes some manual steps
> to setup!

# Why?
Why run LSPs in a Docker container?

Short answer, I wanted to avoid installing NodeJS and NPM running on my local
machine globally. It's merely for security reasons. I want to keep my machines
clean from anything that can be used even by me to install something that I
don't want, given the fact that NPM was recently used as a convenient tool by
hackers. Sha1-Hulud! Remember that?!

Now that you know the WHY, let's now do the HOW!

### Step 1
* Clone this repository.
* Run `docker compose build` to build the images.

After the images have been successfully built we've to now make some changes to
the configuration of each LSP so we can setup proper commands for spinning up
a service for each LSP.

### Step 2
Initially I created some bash script wrappers for triggering a Docker service
but soon realized it was unnecessary extra work.

For example for TypeScript I had the following script which I had inside
`~/.local/share/nvim/bin/lsp/typescript-language-server` along with scripts for
other LSPs.

```bash
#!/bin/bash

docker run --rm -i \
  -v "$PWD":"$PWD" \
  -w "$PWD" \
  local-nodelsps:latest \
  typescript-language-server --stdio "$@"
```

However we can avoid this by putting these commands inside Neovim configuration
files. See the exact configuration for TypeScript below:

```lua
-- ~/.config/nvim/after/lsp/ts_ls.lua
return {
  cmd = function(dispatchers, config)
    -- Dynamically resolve the current working directory to map "$PWD"
    local pwd = vim.uv.cwd() or vim.fn.getcwd()

    -- Define the container execution command natively
    local cmd = {
      "docker", "run", "--rm", "-i",
      "-v", pwd .. ":" .. pwd,
      "-w", pwd,
      "local-nodelsps:latest",
      "typescript-language-server", "--stdio"
    }

    -- Fire up the RPC client directly with the array
    return vim.lsp.rpc.start(cmd, dispatchers)
  end,

  before_init = function(params, config)
    params.processId = vim.NIL
  end
}
```

You'll usually put these configurations inside `.config/nvim/after/lsp/ts_ls.lua`.

Configurations for other LSPs:
* CSS
    ```lua
    -- ~/.config/nvim/after/lsp/cssls.lua
    return {
      cmd = function(dispatchers, config)
        -- Dynamically get the current working directory to replicate "$PWD"
        local pwd = vim.uv.cwd() or vim.fn.getcwd()

        -- Define the raw docker execution array
        local cmd = {
          "docker", "run", "--rm", "-i",
          "-v", pwd .. ":" .. pwd,
          "-w", pwd,
          "local-nodelsps:latest",
          "vscode-css-language-server", "--stdio"
        }

        -- Start the RPC client using the Lua table directly
        return vim.lsp.rpc.start(cmd, dispatchers)
      end,
      
      before_init = function(params, config)
        params.processId = vim.NIL
      end
    }
    ```

* GraphQL
    ```lua
    -- ~/.config/nvim/after/lsp/graphql.lua
    -- Dynamically get the current working directory to replicate "$PWD"
    local pwd = vim.uv.cwd() or vim.fn.getcwd()

    return {
      cmd = {
        "docker", "run", "--rm", "-i",
        "-v", pwd .. ":" .. pwd,
        "-w", pwd,
        "local-nodelsps:latest",
        "graphql-lsp", "server", "-m", "stream" -- Split into individual arguments
      },
      
      filetypes = { 'graphql', 'javascript', 'javascriptreact', 'typescript', 'typescriptreact' },

      before_init = function(params, config)
        params.processId = vim.NIL
      end
    }
    ```

* PHP - Phpactor
    ```lua
    -- ~/.config/nvim/after/lsp/phpactor.lua
    -- Dynamically resolve host variables ($PWD and $HOME)
    local pwd = vim.uv.cwd() or vim.fn.getcwd()
    local home = vim.fn.expand('~')

    return {
      cmd = {
        "docker", "run", "--rm", "-i",
        "-v", pwd .. ":" .. pwd,
        "-v", home .. "/.cache/phpactor:/root/.cache/phpactor",
        "-w", pwd,
        "local-phpactor:latest",
        "language-server" -- Appends the original initialization argument natively
      },
      
      -- Added to prevent container PID collisions
      before_init = function(params, config)
        params.processId = vim.NIL
      end
    }
    ```

These are all the LSPs I use for development and as of now these are the only
ones that come with this setup. However you can install other LSPs yourself and
configure them as shown above!

### Step 3 - Test
Open a PHP file e.g. `nvim index.php` and you'll see that PHP diagnosis is
working code suggestions are available.

Also run `docker container ls` so you can see that `local-phpactor:latest` or
`local-nodelsps:latest` is up and running or both of them. If you close
Neovim and run `docker container ls` again you'll see that that container is
down!
