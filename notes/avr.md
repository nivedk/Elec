
**The USART is fully compatible with the AVR UART regarding:**
* Bit locations inside all USART Registers.
* Baud Rate Generation.
* Transmitter Operation.
* Transmit Buffer Functionality.
* Receiver Operation.
**but compatibility may be affected in some special cases**

* The receiving buffer has two buffer registers that act in FIFO format hence UDR must be rad only once for each incoming data
* Status bit must be read before the UDR register is ready
    * because error flags and the 9th data bit are buffered in the receive buffer **`NOT CLEAR`**

### Clock Generation

* Baud Rate genetator clock output = fosc/(UBRR+1)
* The Transmitter divides the baud rate generator clock output by 2, 8 or 16 depending on mode
* The transfer rate can be doubled by setting the U2X bit in UCSRA.
* Setting this bit only has effect for the asynchronous operation. This bit is set this bit to zero when using synchronous operation.
  * the receiver will in this case only use half the number of samples (reduced from 16 to 8) for data sampling and clock recovery, and therefore a more accurate baud rate setting and system clock are required when this mode is used **`NOT CLEAR`**
* AN externas clock can be used in case of synchrnous slave communication. However, the process takes up two clock periods, hence, the external clock frequency must be lesser than Focs/4
* If UCPOL is set, the data will be changed at falling XCK edge and sampled at rising XCK edge.

### Frame format

USART accepts thirty combinations of the following as valid frame formats
* 1 start bit
* 5, 6, 7, 8, or 9 data bits
* no, even or odd parity bit
* 1 or 2 stop bits

![alt text](https://github.com/nivedk/Elec/blob/master/notes/usartframe_format.png")



### UCSRA: USART Control and Status Register A (8-bit)

#### **Bit 7: RxC – USART Receive Complete Flag:**
* one when there is unread data in the receive buffer
* can also be used to generate a Receive Complete Interrupt

#### **Bit 6: TxC – USART Transmit Complete Flag:**
* entire frame in the shift register is shifted out and there is no new data
* can be cleared by writing *a one* to its bit location
* The TXC Flag can generate a Transmit Complete Interrupt

#### **Bit 5: UDRE – USART Data Register Empty:**
* indicates if the transmit buffer (UDR) is ready to receive new data
* If it is one -> buffer is empty i.e ready to be written
* UDRE is set (1) to indicate that the transmitter is ready

#### **Bit 4: FE – Frame Error:**
* When there is an error in the frame, eg stop bit is 1 etc.
* This bit must be set to 0 when writing to UCRSA

#### **Bit 3: DOR – Data Overrun Error:**
* When receive buffer is full and new start bit is detected
* Valid until UDR is ready
* Set to 0 when writing ot UCRSA

#### **Bit 2: PE – Parity Error:**
* Set if there is a parity error **`NOT CLEAR`**
* Valid until UDR is ready
* Set to 0 when writing ot UCRSA

#### **Bit 1: U2X – Double Transmission Speed:**

#### **Bit 0: MPCM – Multi-Processor Communication Mode:**
* When a receiver is listening to more than one transmitter hence must see the address info to extract thecorrect information

**For UCRSB and UCRSC, refer to [this](http://maxembedded.com/2013/09/the-usart-of-the-avr/#Reg)**

### **Initializing USART**

```c
void USART_Init( unsigned int baud ){
  /* Set baud rate */
  UBRRH = (unsigned char)(baud>>8);
  UBRRL = (unsigned char)baud;
  /* Enable receiver and transmitter */
  UCSRB = (1<<RXEN)|(1<<TXEN);
  /* Set frame format: 8data, 2stop bit */
  UCSRC = (1<<URSEL)|(1<<USBS)|(3<<UCSZ0);
}
```

**Note :** If URSEL is zero during a write operation, the UBRRH value will be updated. If URSEL is one, the UCSRC setting will be updated. (*i.e: even if we aretrying to update UBBRH, if USREL is 1, then UCSRC will be updated*) **`NOT CLEAR`** (*Then how are we editing UCSRC as shown in the last line??*)

```c
/*
 * nived_USart.c
 *
 * Created: 7/3/2017 10:32:18 PM
 * Author : Nived
 */
#define F_CPU 14745600UL // Clock Speed
#define BAUD 9600
#include <avr/io.h>
#include <util/delay.h>
#define MYUBRR FOSC/16/BAUD-1

void USART_Init( unsigned int ubrr){
	/*Set baud rate */
	UBRRH = (unsigned char)(ubrr>>8);
	UBRRL = (unsigned char)ubrr;
	/*Enable receiver and transmitter */
	UCSRB = (1<<RXEN)|(1<<TXEN);
	/* Set frame format: 8data, 2stop bit */
	UCSRC = (1<<URSEL)|(1<<USBS)|(3<<UCSZ0);
}

void USART_Transmit( unsigned char data )
{
	/* Wait for empty transmit buffer */
	while ( !( UCSRA & (1<<UDRE)) );
	/* Put data into buffer, sends the data */
	UDR = data;
}

unsigned char USART_Receive( void )
{
	/* Wait for data to be received */
	while ( !(UCSRA & (1<<RXC)) );
	/* Get and return received data from buffer */
	return UDR;
}

void main( void )
{
	USART_Init(95);
	while(1){

		USART_Transmit('a');
		_delay_ms(100);
		//char r = USART_Receive()
	}
}



```



### Full forms
* **UDR -** USART data register
* **UCSRA -** USART Control and Status Register A
* **UBBR -** USART Baud Rate Register
