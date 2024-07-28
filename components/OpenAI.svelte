<script>
    import { Query } from "@evidence-dev/sdk/usql";
    import { QueryLoad } from "@evidence-dev/core-components";
    import OpenAI from "./_OpenAI.svelte";

    export let data = undefined;

    const initialHash = Query.isQuery(data) ? data.hash : undefined;
    let isInitial = data?.hash === initialHash;
    $: isInitial = data?.hash === initialHash;
</script>

<QueryLoad {data} let:loaded>
    <OpenAI
        data={Query.isQuery(loaded) ? Array.from(loaded) : loaded}
        {...$$restProps}
    />
</QueryLoad>
