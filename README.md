# luautricks
A curated list of Luau tricks and/or optimizations

__Table of contents__:
* [Tricks](#tricks)
* [Optimizations](#optimizations)
* [Contributing](#contributing)

## Tricks
1. denesting via `repeat`
>    ```luau
>    -- traditional
>    local folder = workspace:FindFirstChild("PartFolder")
>    for _, child in folder:GetChildren() do
>        local path = child:GetFullName()
>        if path:find("secret") then
>            local second = path:match("secret(.+)")
>            if not second:match("^1$") then
>                local evenmorenesting = second:sub(1, 4)
>                if #evenmorenesting > 2 then
>                    child:Destroy()
>                end
>            end
>        end
>    end
>
>    -- new
>    local folder = workspace:FindFirstChild("PartFolder")
>    for _, child in folder:GetChildren() do
>        repeat
>            local path = child:GetFullName()
>            if not path:find("secret") then
>                break
>            end
>            local second = path:match("secret(.+)")
>            if second:match("^1$") then
>                break
>            end
>            local evenmorenesting = second:sub(1, 4)
>            if not (#evenmorenesting > 2) then
>                break
>            end
>
>            child:Destroy()
>        until true
>    end
>    ```
Both compile to the same bytecode when optimizationLevel > 0.

## Optimizations
### General optimizations
1. `table.concat` over string concatenation
>    ```luau
>    -- slow
>    local result = ""
>    for _, obj in workspace:GetChildren() do
>        result ..= obj.Name
>    end
>
>    -- fast
>    local t_result = {}
>    for _, obj in workspace:GetChildren() do
>        t_result[#t_result + 1] = obj.Name
>    end
>    local result = table.concat(t_result)
>    ```
This can be quite a big optimization because string concatention creates entire string copies whereas `table.concat` uses an internal buffer.

When testing the above with 1,000 instances in my Roblox clone, I experienced a 33.33% decrease in execution time.

### Micro optimizations
These require more extreme cases to notice a difference
1.  alternative `table.insert`
>    ```luau
>    -- slower
>    local t = {}
>    for i = 1, 10000000 do
>        table.insert(t, math.random())
>    end
>
>    -- faster
>    local t = {}
>    for i = 1, 10000000 do
>        t[#t + 1] = math.random()
>    end
>    ```
2.  use arr values exclusively
>    ```luau
>    -- slower
>    local t = { flags = { option = true }, writes = {}, reads = {} }
>    t.flags.option = false
>    if t.flags.option then game:FindService("Players").LocalPlayer:Kick() end
>
>    -- faster
>    local t = { [1] = { [1] = true }, [2] = {}, [3] = {} }
>    t[1][1] = false
>    if t[1][1] then game:FindService("Players").LocalPlayer:Kick() end
>    ```
Tables' array and dictionary capabilities are implemented as two entirely separate things in Luau, and the array portion is inheritly faster if used correctly.

This is, however, extremely tedious and horrible for maintainability, although [moonveil](https://moonveil.cc) offers a [macro](https://moonveil.cc/docs/macros/mv_index_to_num/) if you're only working with one file.

## Contributing
To contribute, either:
1. Reach out to me on discord ([@techhog or userid 402264559509045258](https://discord.com/users/402264559509045258)), or
2. Do so directly via [pull requests](../../pulls); note this is the less prefered option as I am very opinionated and will likely end up heavily modifying your contribution so I would rather you not spend time writing markdown just for me to change everything. 
