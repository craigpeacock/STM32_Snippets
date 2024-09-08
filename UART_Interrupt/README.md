## UART_Interrupt Example

This example targets the [NUCLEO-F411RE](https://www.st.com/en/evaluation-tools/nucleo-f411re.html) Development Board.

The console uses UART2 (TX PA2, RX PA3) that is redirected to the STLink Virtual Com Port. The baud rate is 115,200.

The UART Interrupt component uses UART1 (TX PA9, RX PA10) at 115,200 8N1.

### Sending 

To send data via Interrupts, the HAL_UART_Transmit_IT API call is used:

```
	if (HAL_UART_Transmit_IT(&huart1, (uint8_t*)buffer, len) == HAL_OK)
	{
		printf("Successfully sent %d bytes\r\n", len);
	} else {
		printf("Error sending %d bytes\r\n", len);
	}
```

### Receiving

To being to receive data, the HAL_UARTEx_ReceiveToIdle_IT is called from main. A buffer of 100 bytes is allocated and used by the API to return the received data.

The ReceiveToIdle() call will allow variable length data to be received. A callback will occur upon completion (100 bytes of data received) or an IDLE event. 

```
uint8_t buffer[100];

HAL_UARTEx_ReceiveToIdle_IT(&huart1, buffer, sizeof(buffer));
```

A callback is then created to handle the interrupt. This can be called upon completion, half completion (50 bytes in this case, half of the allocated 100 byte buffer) or idle. To prevent the buffer being displayed at half completion and repeated at full completion, we obtain the event type and filter out the half completed event.

```
void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef* huart, uint16_t size)
{
	HAL_UART_RxEventTypeTypeDef EventType = HAL_UARTEx_GetRxEventType(huart);
	// This call back will also receive halt transfer events (HAL_UART_RXEVENT_HT).
	// Filter these requests out so the buffer is not printed twice.
	if ((EventType == HAL_UART_RXEVENT_IDLE) || (EventType == HAL_UART_RXEVENT_TC)) {
		printf("Received %hu byte(s): %.*s\r\n", size, size, buffer);
		HAL_UARTEx_ReceiveToIdle_IT(huart, buffer, sizeof(buffer));
	}
}
```
