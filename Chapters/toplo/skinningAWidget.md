## Skinning a simple widget

In this chapter we will show how we can take the simple input widget developed in a previous chapter and make it skinnable. 
Remember that we want to create a widget as shown in Figure *@inputFinalSkin@*.

![An integer input widget. % anchor=inputFinalSkin&width=50](figures/input.png )


### Getting started

If you implemented the widget as presented earlier, just copy the class giving it a new name for example `ToNumberInputElement`.
The definition of the `BlNumberInputElement` is available SD:DefineAPlace.

The first thing that we should do is to make `ToNumberInputElement` inherit from `ToElement` as follows:

```
ToElement << #ToNumberInputElement
```

### Define a skin

We define a skin 

```
ToRawSkin << #ToInputElementSkin

```

We will now define action that should be done when the skin is installed. 

```
ToInputElementSkin >> installLookEvent: anEvent
```

```

ToInputElementSkin >> pressedLookEvent: anEvent
```


SD: Je n'ai pas compris comment on passe du skin au theme. 
Ou sont definis les token color-border et autre. 

In the `ToNumberInputElement` we define the method 


```
newRawSkin
```

SD: comment on sait ou est defini #'color-border'

SD:Alain est ce que cela fait du sens d'invoker  `defaultSkin:` dans l'initialize?

```
ToNumberInputElement >> initialize
```


Alexis le faisait ailleurs mais je trouvais cela un peu moche:


```
ToNumberInputElement >> openInput: anInput
```


### Define a theme

we also define a theme

```
ToRawTheme << #ToInputElementTheme
```


```
ToInputElementTheme class >> defaultTokenProperties
```


```
ToNumberInputElement class >> openInputWithSkin
```

Why the following does not use the install skin and it does not work. 

the following does call default skin but the button do not get color changed :(

```
openInputWithSkin
```
