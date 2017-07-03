
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

## Clock Generation


## UCSRA: USART Control and Status Register A (8-bit)

### **Bit 7: RxC – USART Receive Complete Flag:**
* one when there is unread data in the receive buffer
* can also be used to generate a Receive Complete Interrupt

### **Bit 6: TxC – USART Transmit Complete Flag:**
* entire frame in the shift register is shifted out and there is no new data
* can be cleared by writing *a one* to its bit location
* The TXC Flag can generate a Transmit Complete Interrupt

### **Bit 5: UDRE – USART Data Register Empty:**
* indicates if the transmit buffer (UDR) is ready to receive new data
* If it is one -> buffer is empty i.e ready to be written
* UDRE is set (1) to indicate that the transmitter is ready

### **Bit 4: FE – Frame Error:**
*


## Full forms
* **UDR -** USART data register
* **UCSRA -** USART Control and Status Register A
* **UBBR -** USART Baud Rate Register
