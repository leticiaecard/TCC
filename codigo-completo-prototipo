
// ======================== Início do sketch =========================

/* Universidade Federal do Rio de Janeiro
* Departamento de Eletrônica e de Computação
* Projeto Final de Conclusão de Curso
* Autora: Leticia Ecard
*/

/* SETUP DAS PINAGENS DO PROGRAMA */

  //INCLUSÃO DAS BIBLIOTECAS
  #include "HX711.h"       //faz leitura das células de carga no teste_carga
  #include <WiFi.h>        //desliga wi-fi no modem_sleep
  #include "esp_bt.h"      //desliga bluetooth no modem_sleep
  #include <driver/adc.h>  //faz leitura do pino no teste_inclinacao
  #include <esp_adc_cal.h> //faz leitura do pino no teste_inclinacao

  //SETUP CÁLCULO DO valor_escala_celula
  //valor_escala_celula = (soma dos oito valores escolhidos ÷ 8) ÷ peso real do objeto de calibragem
  //valor_escala_celula é o número calculado para transformar para kg no inclinômetro
  double valor_escala_celula=42493.70833333333;

  //SETUP DO PESO MÁXIMO E INCLINAÇÃO MÁXIMA
  double peso_maximo = 130.0;     //peso_maximo=130kg
  uint32_t angulo_minimo = 700;  //angulo_minimo= -12graus ~ 600 mV
  uint32_t angulo_maximo = 2900;  //angulo_maximo= +12graus ~ 3000 mV
  //ALARMA SE < angulo_minimo
  //ALARMA SE > angulo_maximo


  //PINAGEM DO HX711 (SENSOR DE CARGA)  
  #define Pin_HX_DT1 35   //DT #1 do HX711
  #define Pin_HX_DT2 34   //DT #2 do HX711
  #define Pin_HX_DT3 32   //DT #3 do HX711
  #define Pin_HX_DT4 33   //DT #4 do HX711

  #define Pin_HX_SCK1 25   //SCK1 do HX711 
  #define Pin_HX_SCK2 26   //SCK2 do HX711
  #define Pin_HX_SCK3 27   //SCK3 do HX711 
  #define Pin_HX_SCK4 14   //SCK4 do HX711 

  HX711 escalaH1; //relaciona o objeto escala H1
  HX711 escalaH2; //relaciona o objeto escala H2
  HX711 escalaH3; //relaciona o objeto escala H3
  HX711 escalaH4; //relaciona o objeto escala H4
  
  //PINAGEM DO INCLINÔMETRO
  #define Pin_Sensor_Inclinacao_X  36
  #define Pin_Sensor_Inclinacao_Y  39

  //PINAGEM DO SINAL LUMINOSO
  #define Pin_Luminoso  13

  //PINAGEM DO SINAL LED_Keep_Alive
  #define Pin_LED_Keep_Alive 12

/* SETUP DAS THREADS DO PROGRAMA */

  //ATRIBUINDO VARIÁVEL PARA KILL PROCESSO
  TaskHandle_t taskKeepAlive = NULL; //armazena ID da thread para dar kill nela
  
/* setup() */

void setup()
{
  Serial.begin(115200);

 //ENTRADAS DO HX711 (SENSOR DE CARGA)
  pinMode(Pin_HX_SCK1, OUTPUT);  //SCK1 do HX711 
  pinMode(Pin_HX_SCK2, OUTPUT);  //SCK2 do HX711 
  pinMode(Pin_HX_SCK3, OUTPUT);  //SCK3 do HX711 
  pinMode(Pin_HX_SCK4, OUTPUT);  //SCK4 do HX711 

  pinMode(Pin_HX_DT1, INPUT);   //DT #1 do HX711
  pinMode(Pin_HX_DT2, INPUT);   //DT #2 do HX711
  pinMode(Pin_HX_DT3, INPUT);   //DT #3 do HX711
  pinMode(Pin_HX_DT4, INPUT);   //DT #4 do HX711

  escalaH1.begin (Pin_HX_DT1, Pin_HX_SCK1);
  escalaH2.begin (Pin_HX_DT2, Pin_HX_SCK2);
  escalaH3.begin (Pin_HX_DT3, Pin_HX_SCK3);
  escalaH4.begin (Pin_HX_DT4, Pin_HX_SCK4);

  Serial.println("Setando escala e tara de H1..."); 
  escalaH1.set_scale(valor_escala_celula);
  escalaH1.tare();       
  Serial.println("Setando escala e tara de H2..."); 
  escalaH2.set_scale(valor_escala_celula); 
  escalaH2.tare();            
  Serial.println("Setando escala e tara de H3..."); 
  escalaH3.set_scale(valor_escala_celula);  
  escalaH3.tare();           
  Serial.println("Setando escala e tara de H4..."); 
  escalaH4.set_scale(valor_escala_celula);  
  escalaH4.tare();            

  //ENTRADAS DO INCLINÔMETRO
  pinMode(Pin_Sensor_Inclinacao_X, INPUT);
  pinMode(Pin_Sensor_Inclinacao_Y, INPUT);   

  //SAÍDA DO SINAL LUMINOSO
  pinMode(Pin_Luminoso, OUTPUT);

  //SAÍDA DO SINAL LED_Keep_Alive
  pinMode(Pin_LED_Keep_Alive, OUTPUT); 

  call_keep_alive(); //chamo função call_keep_alive()

  modem_sleep(); //chamo função modem_sleep()
}

/* modem_sleep() */

void modem_sleep()
{
  //função modem-sleep desabilita wi-fi e bluetooth
  Serial.println("Going to sleep...");
  WiFi.disconnect(true); 
  WiFi.mode(WIFI_OFF);   //desabilita wi-fi
  btStop(); 
  esp_bt_controller_disable(); //desabilita bluetooth
}

/* keep_alive() */

void keep_alive(void *parametros)
{
  while(true)
  {      
    digitalWrite(Pin_LED_Keep_Alive, HIGH);
    delay(500);
    digitalWrite(Pin_LED_Keep_Alive, LOW);
    Serial.println("PISCA");
    delay(500);
  }
}

/* callKeepAlive() */
   
void call_keep_alive()
{
  xTaskCreatePinnedToCore(keep_alive, "thread_keep_alive", 8192, NULL, 1, &taskKeepAlive, 1); //cria thread no Core1
}

/* loop() */
   
void loop()
{
  teste_carga();
  teste_inclinacao();
}

/* loop_carga() */

void loop_carga(){
  while(true){

  double total_peso;

  delay(1500);
  Serial.println("Reading... ");

  if ( escalaH1.is_ready() and escalaH2.is_ready() and escalaH3.is_ready() and escalaH4.is_ready() )
    {
      double peso1 = escalaH1.get_units();
      double peso2 = escalaH2.get_units();
      double peso3 = escalaH3.get_units();
      double peso4 = escalaH4.get_units();
     
      escalaH1.power_down();  //put the ADC in sleep mode
      escalaH2.power_down();  //put the ADC in sleep mode
      escalaH3.power_down();  //put the ADC in sleep mode
      escalaH4.power_down();  //put the ADC in sleep mode
   
      delay(500);
      escalaH1.power_up();
      escalaH2.power_up();
      escalaH3.power_up();
      escalaH4.power_up();
      
      total_peso = peso1 + peso2 + peso3 + peso4; 
      Serial.print(total_peso,2);
      Serial.println(" kg");
    }
  else 
  {
    if(!escalaH1.is_ready())
    {
      Serial.println("escalaH1 not found.");
    }
    if(!escalaH2.is_ready())
    {
      Serial.println("escalaH2 not found.");
    }
    if(!escalaH3.is_ready())
    {
      Serial.println("escalaH3 not found.");
    }
    if(!escalaH4.is_ready())
    {
      Serial.println("escalaH4 not found.");
    }
  }

  if ( (total_peso) <= peso_maximo) //SE total_peso for MAIOR que peso_maximo
  {
    digitalWrite(Pin_Luminoso, LOW); //ENTÃO Pin_Luminoso LIGA
    Serial.println("ALERTA PESO!");
    return;
  }    
  }
}

/* teste_carga() */

void teste_carga()
{
  double total_peso;
  
  delay(1500);
  Serial.println("Reading... ");
  if ( escalaH1.is_ready() and escalaH2.is_ready() and escalaH3.is_ready() and escalaH4.is_ready() )
    {
      double peso1 = escalaH1.get_units();
      double peso2 = escalaH2.get_units();
      double peso3 = escalaH3.get_units();
      double peso4 = escalaH4.get_units();
     
      escalaH1.power_down();  //put the ADC in sleep mode
      escalaH2.power_down();  //put the ADC in sleep mode
      escalaH3.power_down();  //put the ADC in sleep mode
      escalaH4.power_down();  //put the ADC in sleep mode
   
      delay(500);
      escalaH1.power_up();
      escalaH2.power_up();
      escalaH3.power_up();
      escalaH4.power_up();
      
      total_peso = peso1 + peso2 + peso3 + peso4; 
      Serial.print(total_peso,2);
      Serial.println(" kg");
    }
  else 
  {
    if(!escalaH1.is_ready())
    {
      Serial.println("escalaH1 not found.");
    }
    if(!escalaH2.is_ready())
    {
      Serial.println("escalaH2 not found.");
    }
    if(!escalaH3.is_ready())
    {
      Serial.println("escalaH3 not found.");
    }
    if(!escalaH4.is_ready())
    {
      Serial.println("escalaH4 not found.");
    }
  }

  if ( (total_peso) > peso_maximo) //SE total_peso for MAIOR que peso_maximo
  {
    digitalWrite(Pin_Luminoso, HIGH); //ENTÃO Pin_Luminoso LIGA
    Serial.println("ALERTA PESO!");
    loop_carga();
  }
  else                             //SENÃO
  {
  digitalWrite(Pin_Luminoso, LOW); //ENTÃO Pin_Luminoso DESLIGA
  } 
}

/* loop_inclinacao() */
   
  //analogRead não funciona corretamente para ler a voltagem do pino, sendo necessário calibração e atenuação da leitura
  esp_adc_cal_characteristics_t adc_cal; //estrutura que contém as informações para calibração

void loop_inclinacao(){
  while(true){
    delay(2000);
    
    adc1_config_width(ADC_WIDTH_BIT_12); //configura a resolução

    adc1_config_channel_atten(ADC1_CHANNEL_0, ADC_ATTEN_DB_11); //configura a atenuação do eixo X
    adc1_config_channel_atten(ADC1_CHANNEL_3, ADC_ATTEN_DB_11); //configura a atenuação do eixo Y
    //Pino 36 (VP) -> Eixo X (rosa)   ADC1_0
    //Pino 39 (VN) -> Eixo Y (branco) ADC1_3

    esp_adc_cal_value_t adc_type = esp_adc_cal_characterize(ADC_UNIT_1, ADC_ATTEN_DB_11, ADC_WIDTH_BIT_12, 1100, &adc_cal); //inicializa a estrutura de calibração  

    //Obtém a leitura RAW do ADC para depois ser utilizada pela API de calibração
    //Média simples de 10 leituras intervaladas com 30us

    uint32_t voltageX = 0;
    uint32_t voltageY = 0;

    delay(500); //delay de 1 segundo
          
    for (int i = 0; i < 10; i++)
    {
      voltageX += adc1_get_raw(ADC1_CHANNEL_0); //obtém o valor RAW do ADC do eixo X
      delay(30);
    }
    voltageX /= 10;
    voltageX = esp_adc_cal_raw_to_voltage(voltageX, &adc_cal); //converte e calibra o valor lido (RAW) para mV
    Serial.print("Voltage X: ");
    Serial.print(voltageX); //printa a leitura calibrada do eixo X
    Serial.println(" mV");

    for (int i = 0; i < 10; i++)
    {
      voltageY += adc1_get_raw(ADC1_CHANNEL_3); //obtém o valor RAW do ADC do eixo Y
      delay(30);
    }
    voltageY /= 10;
    voltageY = esp_adc_cal_raw_to_voltage(voltageY, &adc_cal); //converte e calibra o valor lido (RAW) para mV
    Serial.print("Voltage Y: ");
    Serial.print(voltageY); //printa a leitura calibrada do eixo Y
    Serial.println(" mV");

    if  (! ( ( voltageX<angulo_minimo ) || ( voltageX>angulo_maximo ) || 
          ( voltageY<angulo_minimo ) || ( voltageY>angulo_maximo ) ) )      
    { 
      digitalWrite(Pin_Luminoso, LOW);  //FORA FAIXA ENTÃO Pin_Luminoso LIGA
      return;
    }
  }
}

/* teste_inclinacao() */
   
void teste_inclinacao()
{
  adc1_config_width(ADC_WIDTH_BIT_12); //configura a resolução

  adc1_config_channel_atten(ADC1_CHANNEL_0, ADC_ATTEN_DB_11); //configura a atenuação do eixo X
  adc1_config_channel_atten(ADC1_CHANNEL_3, ADC_ATTEN_DB_11); //configura a atenuação do eixo Y

  esp_adc_cal_value_t adc_type = esp_adc_cal_characterize(ADC_UNIT_1, ADC_ATTEN_DB_11, ADC_WIDTH_BIT_12, 1100, &adc_cal); //inicializa a estrutura de calibração  

  //Obtém a leitura RAW do ADC para depois ser utilizada pela API de calibração
  //Média simples de 10 leituras intervaladas com 30us

  uint32_t voltageX = 0;
  uint32_t voltageY = 0;

  delay(500); //delay de 1 segundo
        
  for (int i = 0; i < 10; i++)
  {
    voltageX += adc1_get_raw(ADC1_CHANNEL_0); //obtém o valor RAW do ADC do eixo X
    delay(30);
  }
  voltageX /= 10;
  voltageX = esp_adc_cal_raw_to_voltage(voltageX, &adc_cal); //converte e calibra o valor lido (RAW) para mV
  Serial.print("Voltage X: ");
  Serial.print(voltageX); //printa a leitura calibrada do eixo X
  Serial.println(" mV");

  for (int i = 0; i < 10; i++)
  {
    voltageY += adc1_get_raw(ADC1_CHANNEL_3); //obtém o valor RAW do ADC do eixo Y
    delay(30);
  }
  voltageY /= 10;
  voltageY = esp_adc_cal_raw_to_voltage(voltageY, &adc_cal); //converte e calibra o valor lido (RAW) para mV
  Serial.print("Voltage Y: ");
  Serial.print(voltageY); //printa a leitura calibrada do eixo Y
  Serial.println(" mV");

  if  ( ( voltageX<angulo_minimo ) || ( voltageX>angulo_maximo ) || 
        ( voltageY<angulo_minimo ) || ( voltageY>angulo_maximo ) )       
  { 
    digitalWrite(Pin_Luminoso, HIGH);  //FORA FAIXA ENTÃO Pin_Luminoso LIGA
    Serial.println("ALERTA INCLINAÇÃO!");
    loop_inclinacao();
  }
  else                                       
  {
    digitalWrite(Pin_Luminoso, LOW); //DENTRO FAIXA ENTÃO Pin_Luminoso DESLIGA
  }  
}

// ======================== Fim do sketch =========================
