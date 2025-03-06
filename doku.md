````C++
void vTaskCodeOn( void * pvParameters )
{
  /* The parameter value is expected to be 1 as 1 is passed in the
     pvParameters value in the call to xTaskCreate() below.
     configASSERT( ( ( uint32_t ) pvParameters ) == 1 );
     */
  for( ;; )
  {
    printf("led-on\t");
    ledPin.set_high();

    nonBlockingDelay(WAIT);

    vTaskPrioritySet( xTaskOffHandle, uxTaskPriorityGet(NULL)+1 );
    printf("priorityOn: %i priorityOff: %i\r\n", int(uxTaskPriorityGet(xTaskOnHandle)), int(uxTaskPriorityGet(xTaskOffHandle)));
  }
}

void vTaskCodeOff( void * pvParameters )
{
  for( ;; )
  {
    printf("led-off\t");
    ledPin.set_low();

    nonBlockingDelay(WAIT);

    vTaskPrioritySet( xTaskOffHandle, uxTaskPriorityGet(NULL)-2 );
    printf("priorityOn: %i priorityOff: %i\r\n", int(uxTaskPriorityGet(xTaskOnHandle)), int(uxTaskPriorityGet(xTaskOffHandle)));
  }
}```

Beim Setzen der neuen Prioritäten springt man sofort in den Task mit der höheren Priorität. Wenn man wieder zurückkommt zu dem Task, von dem man weggewechselt hat, beginnt man nicht von vorne, sondern von dort, wo man weggesprungen ist.
````
