<script>
    import OpenAI from "openai";
    export let data = undefined;
    export let prompt = "Make a chart"
    export let response = "";

    import * as Evidence from "@evidence-dev/core-components";

    let apiKey = undefined;
    let parsedResponse = [];

    let generatedAtLeastOnce = false;
    let generating = false;

    const exampleResponse = [
        { type: "LineChart" , props: {x: "month", y: "sales", series: "product" }}
    ];

    const allowedProps = [
        {
            "x": "column name", 
            "y": "column name", 
            "series": "column name", 
            "labels": "bool", 
            "yGridlines": "bool",
            "yAxisLabels": "bool",
            "title": "string", 
            "xAxisTitle": "string", 
            "yAxisTitle": "string", 
            "yFmt": "excel format code | currencycode",
            "size": "number"
        }
    ];

    const allowedCharts = [ "LineChart", "BarChart", "AreaChart", "ScatterPlot",  "DataTable"];

    const fetchResponse = async () => {
        if (!apiKey) {
            apiKey = window.prompt("Please enter your OpenAI API key:");
            if (!apiKey) {
                response = "API key is required.";
                return;
            }
        }
        generating = true;
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
                        content: `You are an analyst making charts on the following data: ${data}. Respond with a set of components. Use JSON format like this example: ${JSON.stringify(exampleResponse)}. Allow these props but only use [x, y, series] unless the user specifies: ${JSON.stringify(allowedProps)}. Allowed charts: ${allowedCharts}. Do not wrap the reponse in code blocks.`,
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
            generating = false;
        } catch (error) {
            response = error.message;
            generating = false;
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
        return `<${chartName} data=${'{'}data${'}'}\n${Object.entries(item.props).map(([key, value]) => `\t${key}="${value}"`).join("\n")}\n />`;
    };

    const getComponent = (componentName) => {
        return Evidence[componentName];
    };
</script>

<main class="{generating ? 'opacity-50' : ''}">
    {#if !generatedAtLeastOnce}
        <button on:click={fetchResponse} class="w-30 text-white font-bold py-1 px-3 rounded-lg bg-blue-500 hover:bg-blue-700 transition">
            Make me a chart
        </button>    
    {:else}
        <div>
            {#each parsedResponse as item}
                {#if getComponent(item.type)}
                    <svelte:component this={getComponent(item.type)} {data} {...item.props} />
                    <pre class="bg-gray-50 border border-gray-200 rounded p-2 my-2 text-xs"><code>{renderCode(item)}</code></pre>
                {:else}
                    <p>Unknown component: {item.type}</p>
                {/if}
            {/each}
        </div>
        <p>Edit chart prompt</p>
        <div class="flex my-2 gap-2">
            <input type="text" bind:value={prompt} class="w-full border border-gray-300 rounded-lg p-1 px-2" on:keydown={(e) => { if (e.key === "Enter") { fetchResponse(); }}} />
            <button on:click={fetchResponse} class="w-30 text-white font-bold py-1 px-3 rounded-lg bg-blue-500 hover:bg-blue-700 transition">Regenerate</button>
            <button on:click={() => { generatedAtLeastOnce = false, prompt = 'Make a chart' }} class="w-30 text-white font-bold py-1 px-3 rounded-lg bg-red-500 hover:bg-red-700 transition">Reset</button>
        </div>
    {/if}
    
</main>
