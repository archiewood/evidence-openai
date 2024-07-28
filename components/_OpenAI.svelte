<script>
    import OpenAI from "openai";

    export let data = undefined;
    export let prompt = "Briefly describe this data";
    export let response = "Click for summary";

    let apiKey = "";

    const fetchResponse = async () => {
        if (!apiKey) {
            apiKey = window.prompt("Please enter your OpenAI API key:");
            if (!apiKey) {
                response = "API key is required.";
                return;
            }
        }
        response = "Thinking...";
        const client = new OpenAI({
            apiKey,
            dangerouslyAllowBrowser: true,
        });

        try {
            const result = await client.chat.completions.create({
                model: "gpt-4o-mini",
                messages: [
                    {
                        role: "system",
                        content: `You are an analyst providing brief insights on the following data: ${data}. Render response as HTML tags (p ul li b i), every single tag with the class "markdown". Do not wrap the response in markdown fences. Max 3 bullets but less if possible.`,
                    },
                    { role: "user", content: prompt },
                ],
                max_tokens: 250,
            });

            response = result.choices[0].message.content;
            console.log(response);
        } catch (error) {
            response = error.message;
        }
    };
</script>

<main>
    <p class="markdown">{@html response}</p>
    {#if response !== "Click for summary" && response !== "Thinking..."}
        <p>Ask a different question</p>
        <input
            type="text"
            bind:value={prompt}
            class="w-full border border-gray-300 rounded-lg p-1 my-2"
            on:keydown={(e) => {
                if (e.key === "Enter") {
                    fetchResponse();
                }
            }}
        />
    {/if}
    <button
        on:click={fetchResponse}
        class="w-30 text-white font-bold py-1 px-3 rounded-lg {response ===
        'Thinking...'
            ? 'cursor-not-allowed bg-blue-300'
            : 'bg-blue-500 hover:bg-blue-700'} transition"
    >
        {#if response === "Click for summary"}
            Describe
        {:else if response === "Thinking..."}
            Generating
        {:else}
            Regenerate
        {/if}
    </button>
    {#if response !== "Click for summary" && response !== "Thinking..."}
        <button
            on:click={() => {
                response = "Click for summary";
                prompt = "Describe this data";
            }}
            class="w-30 text-white font-bold py-1 px-3 rounded-lg bg-red-500 hover:bg-red-700 transition"
        >
            Reset
        </button>
    {/if}
    
</main>
