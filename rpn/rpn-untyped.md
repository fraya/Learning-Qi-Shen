Calculadora inversa polaca (sin tipos) en Qi
============================================

Después de ojear un poco el libro ["Learn You a Haskell for Great Good!"][1] 
he decidido hacer el [ejemplo][2] sobre una calculadora en [notación inversa 
polaca][3] en Qi, ¡que mala es la envídia!.

Empezaré con el ejemplo en el **lenguaje sin tipos**, que es más sencillo.

Como dice la sagrada Wikipedia:

> La Notación Polaca Inversa, notación de postfijo, o notación posfija, 
> (en inglés, Reverse polish notation, o **RPN**), es un método algebraico 
> alternativo de introducción de datos. Su nombre viene por analogía con 
> la relacionada notación polaca, una notación de prefijo introducida en 1920 
> por el matemático polaco Jan Lukasiewicz, en donde cada operador está antes 
> de sus operandos.

Es decir que para poner **3 + 4** lo hacemos **3 4 +**. Para ello evaluaremos 
los datos directamente cuando se introducen utilizando una pila (o más 
finamente hablando, una estructura LIFO (_Last In First Out_).

Crearemos una función llamada _rpn-aux_ que aceptará una lista con las 
expresiones en notación polaca inversa y una lista que actuará como la pila.

## El algoritmo y su implementación ##

Comencemos a ver el algoritmo de funcionamiento (recordemos que Qi evaluará las 
condiciones por orden):

1. Si las expresiones de entrada son una lista vacía y tenemos un sólo elemento
en la pila, lo devolveremos como resultado.

        #!scheme
        (define rpn-aux
            [] [Resultado] -> Resultado)

Hagamos una prueba:

        #!scheme
        (1-) (rpn-aux [] [3])
        3

Bien pero, ¿que pasaría si dejamos en la pila [3 4]?

        #!scheme
        (2-) (rpn-aux [] [3 4])
        error: partial function rpn-aux;
        track rpn-aux ?  (si(y)/no(n))

El intérprete nos dice que ha probado el primer caso y que no encajaba con los 
datos proporcionados y como no hay otro caso que probar nos aparece un error y 
la posibilidad de depurar la función siguiendo su ejecución. Contestamos _y_ y 
si volvemos a ejecutar la función:

        #!scheme
        (3-) (rpn-aux [] [3 4])
            <1> Inputs to rpn-aux 
            [], [3 4],  ==>
        error: partial function rpn-aux;

En este caso está bastante claro, esperábamos la lista de entrada vacía y un 
sólo parámetro en la pila, al encontrar dos tenemos un error.

2. Si las expresiones de entrada son una lista vacía y tenemos varios 
elementos en la pila, se trata de un error.

        #!scheme
        (define rpn-aux
            [] [Resultado] -> Resultado
            [] [Resultado | Resultados] -> (error "Expresión RPN incorrecta"))

En Qi para definir una lista con un número indeterminado de elementos, 
utilizamos el signo **|**. [_Resultado_ | _Resultados_] indicaría que tenemos 
una lista con un primer elemento (_Resultado_) y a continuación 0 o más 
elementos representados por _Resultados_. 
Si _Resultados_ tuviese 0 elementos (una lista vacía, []) estaríamos en el 
patrón 1 y ya lo habríamos tratado, ya que [_Resultado_ | []] es lo mismo que 
[Resultado]. Por lo tanto en este patrón debemos tener dos o más resultados en 
la pila.  Cargaremos la función de nuevo utilizando 
_(load "nombre-fichero.qi")_

        #!scheme
        (4-) (rpn-aux [] [3])
        3
        (5-) (rpn-aux [] [3 4])
        error: Expresión rpn incorrecta

3. Si el elemento es un operando lo ponemos en la pila

        #!scheme
        (6-) (define rpn-aux 
                [] [Resultado] -> Resultado
                [] [R | Rs] -> (error "Expresión rpn incorrecta"))
                [N | Exprs] Pila -> (rpn-aux Exprs [N | Pila]) where (number? N))

En el último patrón estamos utilizando la guarde _number?_ para asegurarnos de 
que el operando es un número. Para probarlo usaremos la función _track_ y 
seguiremos la ejecución de la función _rpn-aux_

        #!scheme
        (7-) (track rpn-aux)
        rpn-aux
        (8-) (rpn-aux [2] [])
        <1> Inputs to rpn-aux 
            [2], [],  ==>
            <2> Inputs to rpn-aux 
                [], [2],  ==>
            <2> Output from rpn-aux 
                ==> 2
        <1> Output from rpn-aux 
            ==> 2
        2

De acuerdo. Introducimos una expresión con el número 2, este pasa a la pila y 
al estar la lista de expresiones vacía, nos devuelve el único elemento de la 
pila, el 2. Continuemos.

4. Por último supondremos que las funciones que podemos ejecutar son +, -, * 
y /. Todas las operaciones son binarias luego necesitamos dos operandos en la 
pila. Añadiremos la siguiente línea de código:

        #!scheme
        [F | Exprs] [X Y | Z] -> (rpn-aux Exprs [(F Y X) | Z]) where (element? F [+ - * /]))

_F_ será la operación binaria y mediante la guarda nos aseguramos que sea una 
de las funciones que hemos escogido. Nótese también que ejecutamos la función 
_(F Y X)_  cambiando el orden de _Y_ y _X_ ya que los sacamos de una pila. 
Veamos su ejecución:

        #!scheme
        (10-) (rpn-aux [5 1 2 + 4 * + 3 -] [])

        <1> Inputs to rpn-aux 
            [5 1 2 + 4 * + 3 -], [],  ==>
            <2> Inputs to rpn-aux 
                [1 2 + 4 * + 3 -], [5],  ==>
                <3> Inputs to rpn-aux 
                    [2 + 4 * + 3 -], [1 5],  ==>
                    <4> Inputs to rpn-aux 
                        [+ 4 * + 3 -], [2 1 5],  ==>
                        <5> Inputs to rpn-aux 
                            [4 * + 3 -], [3 5],  ==>
                            <6> Inputs to rpn-aux 
                                [* + 3 -], [4 3 5],  ==>
                                <7> Inputs to rpn-aux 
                                    [+ 3 -], [12 5],  ==>
                                    <8> Inputs to rpn-aux 
                                        [3 -], [17],  ==>
                                        <9> Inputs to rpn-aux 
                                            [-], [3 17],  ==>
                                            <10> Inputs to rpn-aux 
                                                [], [14],  ==>
                                            <10> Output from rpn-aux 
                                                ==> 14
                                        <9> Output from rpn-aux 
                                            ==> 14
                                    <8> Output from rpn-aux 
                                        ==> 14
                                <7> Output from rpn-aux 
                                    ==> 14
                            <6> Output from rpn-aux 
                                ==> 14
                        <5> Output from rpn-aux 
                            ==> 14
                    <4> Output from rpn-aux 
                        ==> 14
                <3> Output from rpn-aux 
                    ==> 14
            <2> Output from rpn-aux 
                ==> 14
        <1> Output from rpn-aux 
            ==> 14
        14
  
## Errores ##

Bueno algún queda un caso por tratar, ¿que pasa si introducimos una función 
que no sea +, -, *, /?. Tendríamos un error, obviamente. Añadamos pues una 
última condición que sea aplicable a todos los casos que no hemos tratado:

        #!scheme
        _ _ -> (error "Expresión rpn incorrecta")

## Conclusión ##

Para redondear el programita faltan algunas funciones. El programa completo 
quedaría así:

        #!scheme
        (define rpn-aux 
            [] [Resultado] -> Resultado
            [] [R | Rs] -> (error "Expresión rpn incorrecta")
            [N | Exprs] Pila -> (rpn-aux Exprs [N | Pila]) where (number? N)
            [F | Exprs] [X Y | Z] -> (rpn-aux Exprs [(F Y X) | Z]) where (element? F [+ - * /])
            _ _ -> (error "Expresión rpn incorrecta"))

        (define rpn
            Exprs -> (rpn-aux Exprs []))

        (define calculadora-rpn
            fin -> ok
            comenzar -> (do (output "Introduzca una expresión RPN: ")
                            (output "Resultado: ~A~%" (rpn (input)))
                            (if (y-or-n? "Continuar? ") 
                                (calculadora-rpn comenzar) 
                                (calculadora-rpn fin))))

Un ejemplo de interacción sería:

        #!scheme
        (11-) (calculadora-rpn comenzar)
        Introduzca una expresión RPN: [2 3 +]
        Resultado: 5
        Continuar?  (si(y)/no(n)) y
        Introduzca una expresión RPN: [2 3 4]
        error: Expresión rpn incorrecta

        (12-) (calculadora-rpn comenzar)
        Introduzca una expresión RPN: [2 3 + 5 * 6 - 2 /]
        Resultado: 19/2
        Continuar?  (si(y)/no(n)) n
        ok

Bueno, espero que esta entrada os haya dado una idea de como se hace una 
simple aplicación en Qi. Un saludo.

[1]: http://learnyouahaskell.com
[2]: http://learnyouahaskell.com/functionally-solving-problems#reverse-polish-notation-calculator
[3]: http://es.wikipedia.org/wiki/Notaci%C3%B3n_polaca_inversa

