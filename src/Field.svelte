<script>
    import { getContext, createEventDispatcher } from 'svelte'
    import { values, touched, errors, warnings, validators } from './stores'

    const contextHandleInput = getContext('handleInput')
    const contextHandleBlur = getContext('handleBlur')

    const initialErrors = getContext('initialErrors') || {}
    const initialTouched = getContext('initialTouched') || {}
    const initialValues = getContext('initialValues') || {}
    const initialWarnings = getContext('initialWarnings') || {}

    const sveltikBag = getContext('sveltikBag')

    export let as = undefined
    export let type = 'text'
    export let name
    export let validate = undefined

    const dispatch = createEventDispatcher()

    function handleBlur(e) {
        contextHandleBlur(e)
        dispatch('blur', e)
    }

    function handleInput(e) {
        contextHandleInput(e)
        dispatch('input', e)
    }

    $: validators.update(_v => ({ ..._v, [name]: validate }))
</script>

{#if as === 'select'}
    <select {name} value={$values[name]} {...$$restProps} on:blur={handleBlur} on:input={handleInput}>
        <slot />
    </select>
{:else if as === 'textarea'}
    <textarea {name} value={$values[name]} {...$$restProps} on:blur={handleBlur} on:input={handleInput} />
{:else if as === 'checkbox'}
    <input
        {name}
        type="checkbox"
        checked={$values[name]}
        {...$$restProps}
        on:blur={handleBlur}
        on:change={handleInput}
    />
{:else if typeof as === 'object' || typeof as === 'function'}
    <svelte:component
        this={as}
        field={{ name, value: $values[name], handleBlur: contextHandleBlur, handleInput: contextHandleInput }}
        form={sveltikBag}
        meta={{ initialError: initialErrors[name], initialTouched: initialTouched[name], initialValue: initialValues[name], initialWarning: initialWarnings[name], value: $values[name], touched: $touched[name], error: $errors[name], warning: $warnings[name] }}
        props={$$restProps}
    />
{:else}
    <slot
        field={{ name, value: $values[name], handleBlur: contextHandleBlur, handleInput: contextHandleInput }}
        form={sveltikBag}
        meta={{ initialError: initialErrors[name], initialTouched: initialTouched[name], initialValue: initialValues[name], initialWarning: initialWarnings[name], value: $values[name], touched: $touched[name], error: $errors[name], warning: $warnings[name] }}
    >
        <input {name} {type} value={$values[name]} {...$$restProps} on:blur={handleBlur} on:input={handleInput} />
    </slot>
{/if}
