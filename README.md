CODIGO ARDUINO

const int sensorPin = A0; // Pin conectado al pin Vo del sensor
int sensorValue = 0;
int mappedValue = 0;

void setup() {
  Serial.begin(9600); // Inicia la comunicación serial
}

void loop() {
  // Lee el valor analógico del sensor
  sensorValue = analogRead(sensorPin);

  // Mapea el valor leído al rango de 0 a 127
  mappedValue = map(sensorValue, 0, 600, 0, 127);

  // Imprime el valor mapeado por el puerto serial
  Serial.println(mappedValue);

  delay(100); // Espera un breve período antes de la próxima lectura
}


CODIGO PHYTON

import serial
from pythonosc import udp_client  # Importa la clase UDPClient de la biblioteca python-osc

# Configuración del puerto serie
arduino_port = 'COM4'  # Cambia esto al puerto correcto donde está conectado tu Arduino
baud_rate = 9600

# Configuración de la dirección IP y el puerto de Sonic Pi
sonic_pi_ip = "192.168.1.83"  # Dirección IP de Sonic Pi
sonic_pi_port = 4560  # Puerto para OSC en Sonic Pi

try:
    # Inicialización del puerto serie
    ser = serial.Serial(arduino_port, baud_rate)
    print("Puerto serie abierto")

    # Configuración del cliente OSC para Sonic Pi
    client = udp_client.SimpleUDPClient(sonic_pi_ip, sonic_pi_port)

    while True:
        # Leer los datos del puerto serie
        data = ser.readline().decode().strip()
        # Procesar los datos (en este ejemplo, simplemente imprimirlos)
        print("Datos recibidos:", data)
        # Enviar los datos a Sonic Pi a través de OSC
        client.send_message("/sensor", int(data))  # Envía los datos como un entero
except KeyboardInterrupt:
    # Manejar la interrupción de teclado (Ctrl+C) para cerrar el puerto serie correctamente
    if 'ser' in locals():
        ser.close()
        print("Puerto serie cerrado")

CODIGO SONIC PI

# Configuración del servidor de OSC en Sonic Pi
use_osc "192.168.1.83", 4560

live_loop :osc_piano do
  #ren sync recuerden poner la ruta del dispositivo midi en note_on
  note, velocity = sync "/osc*/sensor"
  #en synth eligen el sintetizador que desean
  synth :sc808_tommid , note: note
end
