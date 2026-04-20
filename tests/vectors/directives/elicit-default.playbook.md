# Elicit with Default

Playbook that exercises the optional default: argument on @elicit for all three types.

## STEP 1: Ask

@elicit(input, "Any extra context?", default:"")
@output(context)

## STEP 2: Confirm

@elicit(confirm, "Proceed?", default:"yes")
@output(go)

## STEP 3: Select

@elicit(select, "Pick a focus", "Cost", "Perf", "DX", default:"Perf")
@output(focus)
