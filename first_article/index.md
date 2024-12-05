# SRAM Pin Format Check Deck

{{< admonition >}}
## Target
> Net Check Pin Name = Terminal Name = Net Name = Label Name
> Label location correction Label~>xy = Pin ~> bBox ~>center
{{< /admonition >}}
## API
{{< highlight markdown >}}
```lisp
;1. Get Pins & Labels: 
geGetSelSet()
cv = geGetEditCellView()
pins = setof(x cv~>shapes x~>purpose=="pin")
labels = setof(x cv~>shapes x~>objType=="label")
textDisplays = setof(x cv~>shapes x~>objType=="textDisplay")
;2. Get Pin Properties
pin_name = pins~>pin~>name
term_name = pins~>pin~>term~>name
pin_center = list(0.5*(caar(pin~>bBox) + caadr(pin~>bBox)) 0.5*(cadar(pin~>bBox) + cadadr(pin~>bBox)))
;3. Label Properties
label_name = label~>name
label_xy = label~>xy
;4. textdisplay Properties
textDisplayname = textDisplays~>parent~>pin~>term~>name
textDisplay_xy = textDisplays~>xy
```
{{< / highlight >}}
## FLOW
{{< mermaid>}}
graph LR

start-- setof func-->L21[Get All textdisplays]
start-- setof func-->L11[Get All Pins]

subgraph 1-Net Check
L11-- if not equal-->L12[modify pin name and 
            terminal name]
end

subgraph 2-Label Location Correction
L21-- if not equal-->L22[fix location]
end
{{< /mermaid>}}
## Debug Summary
1.  New way to find pins
{{< admonition type=tip title="Tip1" open=false >}}
purpose only find pins with layer of pin
cadence suggest find pin with ***shape~>pin property***
{{< link herf = "https://comp.cad.cadence.narkive.com/xvAS01IF/floating-point-rouding-in-skill" >}}
{{< /admonition >}}
2.  Precision problem
{{< admonition type=tip title="Tip1" open=false >}}
float number == may not equal with precsion
make a new func to compare location, **difference < grid is affordable**
{{< /admonition >}}
3.  nil problem
{{< admonition type=tip title="Tip1" open=false >}}
    > - ***Error* plus: can't handle (nil + nil)**
    > - add bBox == nil check
{{< /admonition >}}
## code
{{< highlight markdown >}}
```lisp
(procedure pinNetNameFix(cv)
    (let (pins pin pin_name term_name)
        ;pins = setof(x cv~>shapes x~>purpose=="pin")
        pins = setof(shape geGetEditCellView()~>shapes shape~>pin != nil)
        println("fix netname pin location:")
        (foreach pin pins
            pin_name = pin~>pin~>name
            term_name = pin~>pin~>term~>name
            (if pin_name!=term_name then
                pin~>pin~>name = term_name
                println(pin~>bBox)
            )
        )
        println("fix netname pin finished!")
    )
)
(procedure textDisplayLocationFix(cv)
    (let (variableName)
        textDisplays = setof(x cv~>shapes x~>objType=="textDisplay")
        println("fix label xy location:")
        (foreach textDisplay textDisplays
            textDisplay_xy = textDisplay~>xy
            pin = textDisplay~>parent
            if(pin~>bBox!=nil then
                pin_center = list(0.5*(caar(pin~>bBox) + caadr(pin~>bBox)) 0.5*(cadar(pin~>bBox) + cadadr(pin~>bBox)))
                (if comparexy(textDisplay_xy pin_center)==1 then
                    textDisplay~>xy = pin_center
                    println(pin_center)
                )
            )
        )
        println("fix label xy finished!")
    )
)
(procedure comparexy(xy1 xy2)
    (let (precision x1 y1 x2 y2)
        precision = 10000
        x1 = round(car(xy1)*precision)
        x2 = round(car(xy2)*precision)
        y1 = round(cadr(xy1)*precision)
        y2 = round(cadr(xy2)*precision)
        (if x1==x2 && y1==y2 then
            comparexy = 0
        else
         comparexy = 1
        )
    )
)
cv = geGetEditCellView()
pinNetNameFix(cv)
textDisplayLocationFix(cv)
{{< / highlight >}}

