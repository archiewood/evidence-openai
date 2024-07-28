<script>
    import OpenAI from "openai";
    export let data = undefined;
    export let prompt = "Make a chart"
    export let response = "Click for summary";

    import * as Evidence from "@evidence-dev/core-components";

    let apiKey = undefined;
    let parsedResponse = [];

    let generatedAtLeastOnce = false;

    const exampleResponse = [
        { type: "BarChart" , props: {x: "month", y: "sales", series: "product" }},
    ];

    const allowedProps = ["x", "y", "series", "labels", "title", "subtitle", "xAxisTitle", "yAxisTitle", "yFmt"];

    const allowedCharts = ["BarChart", "LineChart", "AreaChart", "ScatterPlot", "BubbleChart", "Heatmap", "DataTable"];

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
                        content: `You are an analyst making charts on the following data: ${data}. Respond with a list of component names (e.g., BarChart, LineChart and pass in the data object. Use JSON format like this: ${JSON.stringify(exampleResponse)}. Allowed props: ${allowedProps} but only use if the user requests. Allowed charts: ${allowedCharts}. Do not wrap the reponse in code blocks.`,
                    },
                    { role: "user", content: prompt },
                ],
                max_tokens: 250,
            });

            response = result.choices[0].message.content;
            console.log(response);
            generatedAtLeastOnce = true;
            parseResponse(response);
            console.log(parsedResponse);
        } catch (error) {
            response = error.message;
        }
    };

    const parseResponse = (jsonString) => {
        try {
            parsedResponse = JSON.parse(jsonString);
        } catch (e) {
            response = "Failed to parse response";
        }
    };

    const renderCode = (item) => {
        let chartName = item.type;
        return `<${chartName} \n${Object.entries(item.props).map(([key, value]) => `\t${key}="${value}"`).join("\n")}\n />`;
    };

    const getComponent = (componentName) => {
        return Evidence[componentName];
    };
</script>

<main>
    {#if !generatedAtLeastOnce}
        <button on:click={fetchResponse} class="w-30 text-white font-bold py-1 px-3 rounded-lg bg-blue-500 hover:bg-blue-700 transition">
            Make me a chart
        </button>
    {:else if response === "Thinking..."}
        <button class="w-30 text-white font-bold py-1 px-3 rounded-lg bg-blue-300 cursor-not-allowed">Generating</button>
    {:else}
        <div>
            {#each parsedResponse as item}
                {#if getComponent(item.type)}
                    <svelte:component this={getComponent(item.type)} {data} {...item.props} />
                    <pre class="bg-gray-50 border border-gray-200 rounded p-2 my-2"><code>{renderCode(item)}</code></pre>
                {:else}
                    <p>Unknown component: {item.type}</p>
                {/if}
            {/each}
        </div>
        <button on:click={fetchResponse} class="w-30 text-white font-bold py-1 px-3 rounded-lg bg-blue-500 hover:bg-blue-700 transition">Regenerate</button>
        <button on:click={() => { generatedAtLeastOnce = false }} class="w-30 text-white font-bold py-1 px-3 rounded-lg bg-red-500 hover:bg-red-700 transition">Reset</button>
    {/if}
    {#if generatedAtLeastOnce}
        <p>Ask a different question</p>
        <input type="text" bind:value={prompt} class="w-full border border-gray-300 rounded-lg p-1 my-2" on:keydown={(e) => { if (e.key === "Enter") { fetchResponse(); }}} />
    {/if}
</main>
