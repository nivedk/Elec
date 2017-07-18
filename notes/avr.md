
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

![alt text](https://github.com/nivedk/Elec/blob/master/notes/usartframe_format.png)



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
	/*Set baud rate*/
	UBRRH = (unsigned char)(ubrr>>8);
	UBRRL = (unsigned char)ubrr;
	/*Enable receiver and transmitter*/
	UCSRB = (1<<RXEN)|(1<<TXEN);
	/* Set frame format: 8data, 2stop bit*/
  //USBS = 1 means two stop bits
  //URSEL must be 1 when acessing UCRSC
  //(3<<UCSZ0) is the same as (1<<UCSZ1)|(1<<UCZ0)
	UCSRC = (1<<URSEL)|(1<<USBS)|(3<<UCSZ0);
}

void USART_Transmit( unsigned char data )
{
	/* Wait for empty transmit buffer*/
	while ( !( UCSRA & (1<<UDRE)) );
	/* Put data into buffer, sends the data*/
	UDR = data;
}

unsigned char USART_Receive( void )
{
	/* Wait for data to be received*/
	while ( !(UCSRA & (1<<RXC)) );
	/* Get and return received data from buffer*/
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

### SPI - Serial Peripheral Interface

* There can be multiple slaves and masters
* Employs full duplex data transfer. ie: Data is sent and received at the same time
* The SS pin (slave select pin) is used to select the slave, specify its pin modes. If SS is high, all slave SPI pins are normal inputs, will not accept any SPI input, hence SS pin in slave must be held in low state. In case of SS pin of the master, is the SS pin is configured as an output pin, it does not affect the SPI, however, if it is configurted, as an input, it must be held high, otherwise, it will think that another mastr is using it as a slave.

#### Register Description

* SPCR – SPI Control Register - This register is basically the master register i.e. it contains the bits to initialize SPI and control it. Stuff about each bit in the register [here](http://maxembedded.com/2013/11/the-spi-of-the-avr/). Has stuff like data order, SPI enable, master/slave select, clock polarity, clock rate etc.
* SPSR - SPI Status Register The SPI Status Register is the register from where we can get the status of the SPI bus and interrupt flag is also set in this register. Following are the bits in the SPSR register. Has a couple of flags, reserverd bits and a bit for double speed mode.
* SPDR - The SPI Data REgister - The data register.

#### Data modes

* CPOL - Clock Polarity
  * If CPOL is 0, SCK is low when idle
  * If CPOL is 1, SCK is high when idle
* CPHA - Clock Phase
  * When CPHA = 0, data is sampled at clock’s rising/leading edge.
  * When CPHA = 1, data is sampled at clock’s falling/trailing edge.
  * More info [here](http://maxembedded.com/2013/11/serial-peripheral-interface-spi-basics/#CPOL_CPHA)


#### Master code

```c


#define F_CPU 14745600UL // Clock Speed
//#define BAUD 9600

#include <avr/io.h>
#include <util/delay.h>
#include <avr/interrupt.h>


//Initialize SPI Master Device
void spi_init_master (void)
{
    DDRB = (1<<5)|(1<<3);              //Set MOSI, SCK as Output
    SPCR = (1<<SPE)|(1<<MSTR)|(1<<SPR0); //Enable SPI, Set as Master
                                       //Prescaler: Fosc/16, Enable Interrupts
}

//Function to send and receive data
unsigned char spi_tranceiver (char data)
{
    SPDR = data;                       //Load data into the buffer
    while(!(SPSR & (1<<SPIF) ));       //Wait until transmission complete
    return(SPDR);                      //Return received data
}

//Function to blink LED
/*
void led_blink (uint16_t i)
{
    //Blink LED "i" number of times
    for (; i>0; --i)
    {
        PORTD|=(1<<0);
        _delay_ms(100);
        PORTD=(0<<0);
        _delay_ms(100);
    }
}*/

//Main
int main(void)
{
    spi_init_master();                  //Initialize SPI Master
    DDRD |= 0x01;                       //PD0 as Output

    unsigned char data;                 //Received data stored here
    uint8_t x = 0;                      //Counter value which is sent

    while(1)
    {
        data = 0x00;                    //Reset ACK in "data"
        data = spi_tranceiver(++x);     //Send "x", receive ACK in "data"


        /*if(data == ACK) {               //Check condition
            //If received data is the same as ACK, blink LED "x" number of times
            //led_blink(x);
        }
        else {
            If received data is not ACK, then blink LED for a long time so as to determine error
            led_blink(LONG_TIME);
        }*/
        _delay_ms(500);                 //Wait
    }
}



```
#### Slave Code

```c

#define F_CPU 14745600UL // Clock Speed
//#define BAUD 9600

#include <avr/io.h>
#include <avr/interrupt.h>
#include <util/delay.h>

void spi_init_slave (void)
{
    DDRB=(1<<6);                                  //MISO as OUTPUT
    SPCR=(1<<SPE);                                //Enable SPI
}

//Function to send and receive data
unsigned char spi_tranceiver (unsigned char data)
{
    SPDR = data;                                  //Load data into buffer
    while(!(SPSR & (1<<SPIF) ));                  //Wait until transmission complete
    return(SPDR);                                 //Return received data
}

int main(void)
{         

    spi_init_slave();                             //Initialize slave SPI
    unsigned char data, buffer[10];
    DDRA  = 0x00;                                 //Initialize PORTA as INPUT
    PORTA = 0xFF;                                 //Enable Pull-Up Resistors
    while(1)
    {

        data = spi_tranceiver(ACK);               //Receive data, send ACK
                                 //Display received data
        _delay_ms(20);                            //Wait
    }
}



```





### Full forms

* **UDR -** USART data register
* **UCSRA -** USART Control and Status Register A
* **UBBR -** USART Baud Rate Register
* **MISO -** (Master In Slave Out): the input of the Master's shift register, and the output of the Slave's shift register.
* **MOSI -** (Master Out Slave In): the output of the Master's shift register, and the input of the Slave's shift register.
* **SCK -** (Serial Clock): In the Master, this is the output of the clock generator. In the Slave, it is the input clock signal.
