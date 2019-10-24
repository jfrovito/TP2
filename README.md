# Documentación Trabajo Práctico N° 2

**Seminario de Electrónica: Sistemas Embebidos**

**LPC43xx Entradas y Salidas (Digitales) de Propósito General (GPIO) – Diagrama de Estado**

Objetivo:
• Uso del IDE (edición, compilación y depuración de programas).
• Uso de GPIO & Diagrama de Estado (manejo de Salidas y de Entradas Digitales en Aplicaciones).
• Documentar lo que se solicita en c/ítems.

## Uso de Yakindu

Para usar Yakindu se debe ir al directorio:

	firmware_v2/sapi_examples/edu-ciaa-nxp/statecharts

En dicha carpeta se debe buscar el ejemplo o el archivo que se desea simular y compilar. Por ejemplo, para trabajar con un ejemplo, se copia el ejemplo (para no perderlo) y se cambia el nombre a _prefix.sct_. Yakindu va a ejecutar el archivo cuya extensión sea _.sct_. Para simularlo y modificarlo se hace click derecho sobre el archivo _prefix.sct_ y se cliquea en Run As->Statechart Simulation. Para generar el código se preferentemente se borran los archivos que contiene la carpeta _gen_ excepto el _README_, se hace click derecho en el archivo _prefix.sgen_ y se cliquea en _Generate code artifacts_. Se deben generar cuatro archivos dentro de la carpeta _gen_. Luego el compila y debuguea normalmente.

## Estructura de archivos

En la carpeta _statecharts_bare_metal_ se encuentran los archivos _prefix.sct_, _prefix.sgen_, _Makefile_ y los ejemplos, que tienen extensión _.-sct_ para que no se ejecuten.
En la carpeta _scr_ se encuentran los archivos _main.c_ y _TimerTicks.c_. En la carpeta _inc_ se encuentran los archivos _main.h_ y _TimerTicks.h_. En la carpeta _gen_ se encuentran los archivos generados por Yakindu, _Prefix.c_, _Prefix.h_, _PrefixRequired.h_, _sc_types.h_ y _README_.

## Funciones

Se va a empezar analizando el archivo _main.c_. Primero se incluyen los _headers_ necesarios. Luego se definen las opciones de compilación, esto sirve para trabajar con distintos códigos en el mismo archivo. En la siguiente definición se determina qué código se va a ejecutar modificando el paréntesis:

	#define TEST (SCT_1)

Es importante determinar si se van a utilizar eventos de tiempo o no. Esto se realiza modificando la siguiente definición:

	#define __USE_TIME_EVENTS (true)
Se coloca _true_ dentro del paréntesis si se utilizan eventos de tiempo y _false_ si no se utilizan.

Luego se programan algunas de las funciones que están asociadas a las salidas y a las entradas del diagrama de estados. El prototipo de estas funciones es generado por el código de Yakindu, las salidas y entradas del diagrama de estados se vinculan con las salidas y entradas deseadas del firmware mediante estas funciones. Por ejemplo la siguiente función está asociada al encendido o apagado de LEDs:

	void prefixIface_opLED(Prefix* handle, sc_integer LEDNumber, sc_boolean State)
	{
		gpioWrite( (LEDR + LEDNumber), State);
	}

Luego se programa el _main_. Luego de las inicializaciones y configuraciones se entra en un loop dado por un _while(1)_. Cada vez que se dispara la interrupción del _timer_ del sistema _SysTick_ se ejecuta un _if()_. En los ejemplos, dentro de este _if()_ se ejecuta la siguiente función para determinar si fue presionado un botón y retorna qué botón fue presionado:

	uint32_t Buttons_GetStatus_(void) {
		uint8_t ret = false;
		uint32_t idx;

		for (idx = 0; idx < 4; ++idx) {
			if (gpioRead( TEC1 + idx ) == 0)
				ret |= 1 << idx;
		}
		return ret;
	}
Con el siguiente código se vincula la tecla presionada con la función de la entrada del código de Yakindu:

	BUTTON_Status = Buttons_GetStatus_();
	if (BUTTON_Status != 0)									// Event -> evTECXOprimodo => OK
		prefixIface_raise_evTECXOprimido(&statechart, BUTTON_Status);	// Value -> Tecla
	else													// Event -> evTECXNoOprimido => OK
		prefixIface_raise_evTECXNoOprimido(&statechart);

Por último se ejecuta la siguiente función, que corre el ciclo de la lógica generada por Yakindu.

	prefix_runCycle(&statechart);	

La función _prefix_runCycle()_ tiene adentro un _switch()_, que va a hacer que se ejecuten todos los estados que estén activos. En el archivo _Prefix.c_ se programa el resto de las funciones que sirven para la lógica del código y las funciones para ejecutar todos los estados. El resto de los archivos de la carpeta _gen_ contienen las definiciones y los prototipos que usan _Prefix.c_ y _main.c_.
