<script lang="ts">
    import {enhance} from '$lib/form';
    import {scale} from 'svelte/transition';
    import {flip} from 'svelte/animate';
    import type {PageData} from './$types';
    import {browser} from "$app/environment";

    export let data: PageData;
    $: ({jobs} = data)
    const headers = ['Job', 'Box', 'Name', 'Sent', 'Send/Retries', 'File Size', 'Create Time', 'Transfer Rate', 'Last Send/Retry Time']

    const formatBytes = (bytes: number): string => {
        let suffix = ''
        if (bytes > 999999999) {
            bytes /= 1000000000
            suffix = 'GB'
        } else if (bytes > 999999) {
            bytes /= 1000000
            suffix = 'MB'
        } else if (bytes > 999) {
            bytes /= 1000
            suffix = 'KB'
        } else if (bytes)
            suffix = 'Bytes'
        const bytesString = bytes ? bytes.toFixed(1) : '-'

        return `${bytesString} ${suffix}`
    }

    export const formatTransferRate = (rate: number): string => {
        rate *= 8
        let suffix = ''
        if (rate > 999999999) {
            rate /= 1000000000
            suffix = 'Gbps'
        } else if (rate > 999999) {
            rate /= 1000000
            suffix = 'Mbps'
        } else if (rate > 999) {
            rate /= 1000
            suffix = 'Kbps'
        } else if (rate)
            suffix = 'bps'
        const rateString = rate ? rate.toFixed(2) : '-'

        return `${rateString} ${suffix}`
    }

    const getFileName = (path: string): string => {
        const lastIndex = path?.lastIndexOf("\\") || 0
        return path?.substring(lastIndex + 1) || 'filename'
    }

    let rowsPerPage = 15;
    let currentPage = 0;
    $: displayData = [...jobs]

    $: slice = jobs.slice(0, rowsPerPage + 1)

    $: start = currentPage * rowsPerPage
    $: end = Math.min(start + rowsPerPage, displayData.length)
    $: slice = displayData.slice(start, end)
    $: lastPage = Math.max(Math.ceil(displayData.length / rowsPerPage) - 1, 0)
    $: if (currentPage > lastPage)
        currentPage = lastPage
</script>

<svelte:head>
    <title>Todos</title>
    <meta name="description" content="A todo list app"/>
</svelte:head>

<table>
    <thead>
    <tr>
        {#each headers as header}
            <th>{header}</th>
        {/each}
    </tr>
    </thead>
    <tbody>
    {#each slice as {
        id,
        localPath,
        sent,
        retries,
        fileSize,
        createTime,
        transferRate,
        boxId,
        lastRetry
    }, i}
        <tr>
            <td>
                {id}
            </td>
            <td>
                {boxId}
            </td>
            <td>
                {getFileName(localPath)}
            </td>
            <td>
                {sent ? '\u{1f7e2}' : '\u{1f534}'}
            </td>
            <td>
                {retries}
            </td>
            <td>
                {formatBytes(fileSize)}
            </td>
            <td>
                {new Date(createTime).toLocaleDateString()}
            </td>
            <td>
                {formatTransferRate(transferRate)}
            </td>
            <td>
                {new Date(lastRetry).toLocaleDateString()}
            </td>
        </tr>
    {/each}
    </tbody>
</table>
<div style="display: flex; align-items: center; justify-content: space-between">
<button
        on:click={() => currentPage = 0}
        disabled={currentPage === 0}>First Page
</button>
<button
        on:click={() => currentPage--}
        disabled={currentPage === 0}>Prev Page
</button>
<span style="display: inline-block">
    {start + 1}-{end} of {displayData.length}
</span>
<button
        on:click={() => currentPage++}
        disabled={currentPage === lastPage}>Next Page
</button>
<button
        on:click={() => currentPage = lastPage}
        disabled={currentPage === lastPage}>Last Page
</button>
</div>
<style>
    tr:hover{
        background: rgba(0,0,0,0.1);
        transition: all 200ms;
    }
</style>

