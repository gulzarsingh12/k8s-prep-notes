# for loop

## iterate over range 1... 10
`for i in $(seq 1 10); do echo $i; done;`
prints 1 2 3 4 5 6 ... in each line

## iterate over provided value in condition
`for i in hello world 1 10; do echo $i; done;`
prints `hello world 1 10` in each line

# if

## if only
`status="true" && if "$status" == "true"; then echo status is set; fi`

## if else
`status="false" && if "$status" == "true"; then echo status is set; else echo status is not set; fi`

### if else if
`type="color" && if [ "$type" == "color" ]; then echo orange; elif [ "$type" == "hero" ]; then echo spiderman; else echo type not known; fi`
