#include <stdint.h>
#include "stm32l053xx.h"
#include "cmsis_compiler.h"
#include "cmsis_gcc.h"
#include "cmsis_version.h"
#include "core_cm0plus.h"
#include "mpu_armv7.h"
#include "system_stm32l0xx.h"
#if !defined(__SOFT_FP__) && defined(__ARM_FP)
  #warning "FPU is not initialized, but the project is compiling for an FPU. Please initialize the FPU before use."
#endif
//--------------------------------------------------------------------------------------------------------------------------------------------------
  //USART2




/*------------------------------------------------ FUNCIONES------*/
//void delay_ms (uint16_t n);
void USART2_write(uint8_t ch);
void USART2_putstring(uint8_t* StringPtr);
void USART2_putstring_E(uint8_t* StringPtr);
uint8_t USART2_read(void);
/*-------------------------------------------------------------------------------------------- */

uint8_t mychar = 0x00;
//----------------------------------------------------------------------------------------------------------------------------------------------------

#define ESTADO_ESPERANDO_PRIMER_DIGITO 0
#define ESTADO_ESPERANDO_SEGUNDO_DIGITO 1
#define ESTADO_REALIZANDO_OPERACION 2
//--------------------------------------------------------------------------------------------//
void keyboard_config();
void print_bcd_7_segment_decoder_CC(uint8_t);
void delayMs(int n);
void key_pressed();
void init();
void calculador();
uint8_t digito1 = 0;  // variable donde me guarda el primer  valor presionado actual
uint8_t digito2 = 0;   // valor del primer numero nombrado.
uint8_t operacion = 0; // valor del segundo numero nombrado.
//uint8_t estado = 0;
uint8_t estado = ESTADO_ESPERANDO_PRIMER_DIGITO;  // Estado actual de la calculadora
uint8_t val = 0;
uint8_t escrito =0;
int main(void){

	//---------------------------------------------USART2------------------------------------------------------
	/* Enable CLK
		 * For the Peripherals under use via RCC register
		 * Port A
		 * */
		RCC->IOPENR |= 1<<0;
		// Test GPIOA PA5
		GPIOA->MODER &= ~(1<<11);

		// USART 2 Pint Configuracion
		// Alternate Function MODE for PA2 and PA3

		GPIOA->MODER &= ~(1<<4); // PA2 como ALternate FUnction, MODER [10]
		GPIOA->MODER &= ~(1<<6); // PA3 como Alternate FUnction, MODER [10]
		// Select witch MODER, Alternate FUnction 4 para USART2, TX en PA2 :: RX en PA3
		GPIOA->AFR[0] |= 1<<10; // PA2 mapped as AF4
		GPIOA->AFR[0] |= 1<<14; // PA3 mapped as AF4

		// USART2 peripheral configuration
		// USART2 clock enable on RCC APB1ENR register

		RCC->APB1ENR |= 1<<17; // Baud Rate calculation
		USART2->BRR   = 218; // Enable TX and RX para 16mhz
		USART2->CR1  |= (1<<2)|(1<<3); // Enable peripheral
		USART2->CR1  |= 1<<0;

	//--------------------------------------------------------------------------------------------------------------

	init();

	while(1){
		keyboard_config();		//SE MANDA A LLAMAR LA FUNCION QUE TIENE LA CONFIGURACION DE LAS FILAS

		key_pressed();			//SE MANDA A LLAMAR LA FUNCION QUE TIENE LA CONFIGURACION DEL MULTIPLEXADO DE COLUMNAS E IMPRESION EN LOS DISPLAYS

		calculador();      // SE MANDA A LLAMAR LA FUNCION DE CALCULADOR



	}


}


void init(){

	RCC->IOPENR |=0x01<<0;		//HABILITA EL REGISTRO DE RELOJ DEL PUERTO GPIOA
	RCC->IOPENR |=0x01<<1;		//HABILITA EL REGISTRO DE RELOJ DEL PUERTO GPIOB
	RCC->IOPENR |=0x01<<2;		//HABILITA EL REGISTRO DE RELOJ DEL PUERTO GPIOC



	GPIOB->MODER &=~(1<<17);	//PB8 COMO SALIDA CON EL SEGMENTO A
	GPIOB->MODER &=~(1<<5);		//PB2 COMO SALIDA CON EL SEGMENTO B
	GPIOB->MODER &=~(1<<7);		//PB3 COMO SALIDA CON EL SEGMENTO C
	GPIOB->MODER &=~(1<<9);		//PB4 COMO SALIDA CON EL SEGMENTO D
	GPIOB->MODER &=~(1<<13);	//PB6 COMO SALIDA CON EL SEGMENTO E
	GPIOB->MODER &=~(1<<15);	//PB7 COMO SALIDA CON EL SEGMENTO F
	GPIOB->MODER &=~(1<<19);	//PB9 COMO SALIDA CON EL SEGMENTO H

	GPIOA->MODER &=~(1<<1);		//PA0 COMO HABILITADOR DEL COMUN DEL DISPLAY 0
	GPIOA->MODER &=~(1<<3);		//PA1 COMO HABILITADOR DEL COMUN DEL DISPLAY 1

	GPIOA->BSRR = (0x03<<16);	//INICIA COLOCANDO 0 LOS PUERTOS PA0 Y PA1


}
void keyboard_config(){

	//CONFIGURACION DE LAS FILAS
	//CPNFIGURACION DE PC5
	GPIOC->MODER &=~(1<<10);	//SE HABILITA COMO ENTRADA
	GPIOC->MODER &=~(1<<11);
	GPIOC->PUPDR |=(1<<10);		    //SE HABILITA EL PULL-UP DE PC5
	//CONFIGURACION DE PC6
	GPIOC->MODER &=~(1<<12);	//SE HABILITA COMO ENTRADA
	GPIOC->MODER &=~(1<<13);
	GPIOC->PUPDR |=(1<<12);		    //SE HABILITA EL PULL-UP DE PC6
	//CONFIGURACION DE PC8
	GPIOC->MODER &=~(1<<16);	//SE HABILITA COMO ENTRADA
	GPIOC->MODER &=~(1<<17);
	GPIOC->PUPDR |=(1<<16);		     //SE HABILITA EL PULL-UP DE PC8
	//CONFIGURACION DE PC9
	GPIOC->MODER &=~(1<<18);	//SE HABILITA COMO ENTRADA
	GPIOC->MODER &=~(1<<19);
	GPIOC->PUPDR |=(1<<18);		    //SE HABILITA EL PULL-UP DE PC9

	//COLUMN CONFIG
	//PB11 COL0 OUTPUT-HIGH
	GPIOB->MODER &=~(1<<23);	//SE HABILITA COMO ENTRADA
	//PC7 COL1 OUTPUT-HIGH
	GPIOC->MODER &=~(1<<15);	//SE HABILITA COMO ENTRADA
	//PA9 COL2 OUTPUT-HIGH
	GPIOA->MODER &=~(1<<19);  	//SE HABILITA COMO ENTRADA
	//PC4 COL3 OUTPUT-HIGH
	GPIOC->MODER &=~(1<<9);		//SE HABILITA COMO ENTRADA
}

void key_pressed(){
	/*	USART2_putstring_E("Escriba A = suma, B = resta, C = multip, D = div, para mostrar operacion");
				mychar = USART2_read();

				if(mychar == 'f')
				{
					USART2_putstring_E(operacion);
				}
				else if(mychar == 't')
				{
					USART2_putstring_E("Mi apellido Rosales");
				}
				else if(mychar == 'x')
				{
					USART2_putstring_E("No valido");
				}
		---------------------------------------------------------------------------------------------------------*/

	//SCAN COL0
	GPIOB->ODR &=~(1<<11); 	             //COLOCA PB11 COMO 0 PC5
	GPIOC->ODR |=(1<<7);	                 //COLOCA PC7 COMO 1
	GPIOA->ODR |=(1<<9);	                 //COLOCA PA9 COMO 1
	GPIOC->ODR |=(1<<4);	                 //COLOCA PC4 COMO 1
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){			//EVALUANDO FILA 0
		                                                                                             //SE PRESIONO LA TECLA 1

		print_bcd_7_segment_decoder_CC(0x01);	//IMPRIME EL NO. 1
		GPIOA->ODR |=(1<<1);					//HABILITA EL DISPLAY 0
		delayMs(5);								//DELAY
		//GPIOA->BSRR = (0x03<<16);				//RESETEA LOS BITS DE LOS DISPLAYS
	}else if(!(GPIOC->IDR & 0x40)){		//EVALUANDO FILA 1
		                                                                                             //SE PRESIONO LA TECLA 4

		print_bcd_7_segment_decoder_CC(0x04);	//IMPRIME EL NO. 4
		GPIOA->ODR |=(1<<1);					//HABILITA EL DISPLAY 0
		delayMs(5);								//DELAY
		//GPIOA->BSRR = (0x03<<16);				//RESETEA LOS BITS DE LOS DISPLAYS
	}else if(!(GPIOC->IDR & 0x100)){	//EVALUANDO FILA 2
		                                                                                            //SE PRESIONO LA TECLA 7

		print_bcd_7_segment_decoder_CC(0x07);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x200)){	//EVALUANDO FILA 3
	                                                                                             	//SE PRESIONO LA TECLA *
		  if(estado == ESTADO_ESPERANDO_PRIMER_DIGITO){ // Si estamos esperando el primer dígito
		            digito1 = val;  // Almacena el primer dígito
		            estado = ESTADO_ESPERANDO_SEGUNDO_DIGITO;  // Cambia al estado de esperar el segundo dígito
		        }
		        else if(estado == ESTADO_ESPERANDO_SEGUNDO_DIGITO){ // Si estamos esperando el segundo dígito
		            digito2 = val;  // Almacena el segundo dígito
		            estado = ESTADO_REALIZANDO_OPERACION;  // Cambia al estado de realizar operación
		        }


				delayMs(5);

		//GPIOA->ODR |=(1<<1);
		//delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}

	//////////////SCAN COL1
	GPIOB->ODR |=(1<<11);	//COLOCA PB11 COMO 1
	GPIOC->ODR &=~(1<<7);	//COLOCA PC7 COMO 0
	GPIOA->ODR |=(1<<9);	//COLOCA PA9 COMO 1
	GPIOC->ODR |=(1<<4);	//COLOCA PC4 COMO 1
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){			//EVALUANDO FILA 0
                                                                                        //SE PRESIONO TECLA 2
		print_bcd_7_segment_decoder_CC(0x02);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x40)){		//EVALUANDO FILA 1
	                                                                            	//SE PRESIONO LA TECLA 5
			print_bcd_7_segment_decoder_CC(0x05);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x100)){	//EVALUANDO FILA 2
	                                                                            	//SE PRESIONO LA TECLA 8

		print_bcd_7_segment_decoder_CC(0x08);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x200)){	//EVALUANDO FILA 3
	                                                                            	//SE PRESIONO LA TECLA 0

		print_bcd_7_segment_decoder_CC(0x00);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}
	//SCAN COL2
	GPIOB->ODR |=(1<<11);	//COLOCA PB11 COMO 1
	GPIOC->ODR |=(1<<7);	//COLOCA PC7 COMO 1
	GPIOA->ODR &=~(1<<9);	//COLOCA PA9 COMO 0
	GPIOC->ODR |=(1<<4);	//COLOCA PC4 COMO 1
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){			//EVALUANDO FILA 0
	                                                                               	//SE PRESIONO LA TECLA 3

		print_bcd_7_segment_decoder_CC(0x03);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x40)){		//EVALUANDO FILA 1

		print_bcd_7_segment_decoder_CC(0x06);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x100)){	//EVALUANDO FILA 2
		                                                                            //SE PRESIONO LA TECLA 9

		print_bcd_7_segment_decoder_CC(0x09);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x200)){	//EVALUANDO FILA 3
                                                                                                        ////  SE PRESIONO LA TECLA  #


digito1=0;
digito2=0;
estado=0;
				GPIOA->BSRR = (0x03<<16);

		delayMs(5);

	}
	//SCAN COL3
	GPIOB->ODR |=(1<<11);	//COLOCA PB11 COMO 1
	GPIOC->ODR |=(1<<7);	//COLOCA PC7 COMO 1
	GPIOA->ODR |=(1<<9);	//COLOCA PA9 COMO 1
	GPIOC->ODR &=~(1<<4);	//COLOCA PC4 COMO 0
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){			//EVALUANDO FILA 0
		                                                                                //SE PRESIONO LA TECLA   A
		if(estado == ESTADO_REALIZANDO_OPERACION){ // Si estamos listos para realizar la operación
		            // Realiza la operación y almacena el resultado
		            operacion = digito1 + digito2;
		            // También puedes imprimir el resultado en los displays
		            print_bcd_7_segment_decoder_CC(operacion);
		            USART2_putstring_E("operacion suma");     // encabezado de la operacion en la consola
		           // USART2->TDR = 0X34;
		            USART2_write(escrito);       // valor numero escrito en la consola
		        	USART2_write(0x0D); // CR
		        	USART2_write(0x0A); // NL
		            // Vuelve al estado inicial
		            estado = ESTADO_ESPERANDO_PRIMER_DIGITO;
		        }

		delayMs(5);

	}else if(!(GPIOC->IDR & 0x40)){		//EVALUANDO FILA 1
	                                                                                     	//SE PRESIONO LA TECLA B
		if(estado == ESTADO_REALIZANDO_OPERACION){ // Si estamos listos para realizar la operación
			            // Realiza la operación y almacena el resultado
			            operacion = digito2 - digito1;
			            // También puedes imprimir el resultado en los displays
			            print_bcd_7_segment_decoder_CC(operacion);
			            print_bcd_7_segment_decoder_CC(operacion);
			          		            USART2_putstring_E("operacion resta");  // encabezado de la operacion en la consola
			          		          USART2_write(escrito);       // valor numero escrito en la consola
			          		    	  USART2_write(0x0D); // CR
			          		    	 USART2_write(0x0A); // NL
			            // Vuelve al estado inicial
			            estado = ESTADO_ESPERANDO_PRIMER_DIGITO;
			        }
				delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x100)){	//EVALUANDO FILA 2
	                                                                                        	//SE PRESIONO LA TECLA C

		if(estado == ESTADO_REALIZANDO_OPERACION){ // Si estamos listos para realizar la operación
					            // Realiza la operación y almacena el resultado
					            operacion = digito2 * digito1;
					            // También puedes imprimir el resultado en los displays
					            print_bcd_7_segment_decoder_CC(operacion);
					            print_bcd_7_segment_decoder_CC(operacion);
					          		            USART2_putstring_E("operacion multiplicacion");     // encabezado de la operacion en la consola
					          		          USART2_write(escrito);    // valor numero escrito en la consola
					          		       	  USART2_write(0x0D); // CR
					          		    	  USART2_write(0x0A); // NL
					            // Vuelve al estado inicial
					            estado = ESTADO_ESPERANDO_PRIMER_DIGITO;
					        }

		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x200)){	//EVALUANDO FILA 3
		                                                                                                 //SE PRESIONO LA TECLA D
		if(estado == ESTADO_REALIZANDO_OPERACION){ // Si estamos listos para realizar la operación
							            // Realiza la operación y almacena el resultado
							            operacion = digito2 / digito1;
							            // También puedes imprimir el resultado en los displays
							            print_bcd_7_segment_decoder_CC(operacion);
							            print_bcd_7_segment_decoder_CC(operacion);
							          		            USART2_putstring_E("operacion division");    // encabezado de la operacion en la consola
							          		          USART2_write(escrito);         // valor numero escrito en la consola
							          		    	  USART2_write(0x0D); // CR
							          		    	  USART2_write(0x0A); // NL
							            // Vuelve al estado inicial
							            estado = ESTADO_ESPERANDO_PRIMER_DIGITO;
							        }

		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}
}

void print_bcd_7_segment_decoder_CC(uint8_t val){


digito1 = val;                                           ////// // Actualiza el último valor mostrado//

	if(estado == ESTADO_ESPERANDO_PRIMER_DIGITO){ // Si estamos esperando el primer dígito
			            digito1 = val;  // Almacena el primer dígito
			            estado = ESTADO_ESPERANDO_SEGUNDO_DIGITO;  // Cambia al estado de esperar el segundo dígito
			        }
			        else if(estado == ESTADO_ESPERANDO_SEGUNDO_DIGITO){ // Si estamos esperando el segundo dígito
			            digito2 = val;  // Almacena el segundo dígito
			            estado = ESTADO_REALIZANDO_OPERACION;  // Cambia al estado de realizar operación
			        }


	if(val==0x00){						//ENCIENDE EL NUMERO 0 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		escrito = 0x30;
	}else if(val==0x01){				//ENCIENDE EL NUMERO 1 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		escrito = 0x31;
	}else if(val==0x02){				//ENCIENDE EL NUMERO 2 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		escrito = 0x32;
	}else if(val==0x03){				//ENCIENDE EL NUMERO 3 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		escrito = 0x33;
	}else if(val==0x04){				//ENCIENDE EL NUMERO 4 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		escrito = 0x34;
	}else if(val==0x05){				//ENCIENDE EL NUEMRO 5 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		escrito = 0x35;
	}else if(val==0x06){				//ENCIENDE EL NUEMRO 6 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		escrito = 0x36;
	}else if(val==0x07){				//ENCIENDE EL NUEMRO 7 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		escrito = 0x37;
	}else if(val==0x08){				//ENCIENDE EL NUEMRO 8 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		escrito = 0x38;
	}else if(val==0x09){				//ENCIENDE EL NUMERO 9 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		escrito = 0x39;
	}else if(val==0x0A){				//ENCIENDE LA LETRA A
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		escrito = 0x41;
	}else if(val==0x0B){				//ENCIENDE LA LETRA b
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		escrito = 0x42;
	}else if(val==0x0C){				//ENCIENDE LA LETRA C
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		escrito = 0x43;
	}else if(val==0x0D){				//ENCIENDE LA LETRA d
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		escrito = 0x44;
	}
}

void delayMs(int n){		//DELAY EXPERIMENTAL
	int i;
	for(; n > 0; n--)
		for(i = 0; i < 240; i++);
}

void calculador(){
	if (estado == 'A')
		operacion = digito1 + digito2;
	if(estado == 'B')
		operacion = digito1 - digito2;
	if (estado =='C')
		operacion = digito1 * digito2;
	if (estado =='D')
			operacion = digito1 / digito2;
}


uint8_t USART2_read(void)
{
	while(!(USART2->ISR & 0x0020)){}
	return USART2->RDR ;

}




void USART2_write(uint8_t ch)
{
	while (!(USART2->ISR & 0x0080)){}
	USART2->TDR = ch;
}



void USART2_putstring(uint8_t* StringPtr)
{
	while(*StringPtr != 0x00)
	{
		USART2_write(*StringPtr);
		StringPtr++;
	}
}


void USART2_putstring_E(uint8_t* StringPtr)
{
	while(*StringPtr != 0x00)
	{
		USART2_write(*StringPtr);
		StringPtr++;
	}
	USART2_write(0x0D); // CR
	USART2_write(0x0A); // NL
}

