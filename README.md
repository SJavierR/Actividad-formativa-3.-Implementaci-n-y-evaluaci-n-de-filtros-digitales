import numpy as np  

import matplotlib.pyplot as plt from scipy  

import signal  

from scipy.fft import fft, fftfreq 

Configuración inicial 

plt.style.use('seaborn-v0_8') np.random.seed(42) # Para reproducibilidad 

1. Crear señales de prueba 

def generar_senal_compuesta(frecuencias, duracion=1.0, fs=1000): """ Genera una señal compuesta por múltiples frecuencias con ruido blanco 

Args: 
    frecuencias (list): Lista de frecuencias en Hz 
    duracion (float): Duración en segundos 
    fs (int): Frecuencia de muestreo 
     
Returns: 
    tuple: (tiempo, señal) 
""" 
t = np.linspace(0, duracion, int(fs * duracion), endpoint=False) 
senal = np.zeros_like(t) 
 
for f in frecuencias: 
    senal += np.sin(2 * np.pi * f * t) 
 
# Añadir ruido blanco 
ruido = 0.5 * np.random.normal(size=len(t)) 
senal_ruidosa = senal + ruido 
 
return t, senal, senal_ruidosa 
  

Parámetros de la señal 

frecuencias = [10, 50, 100, 200] # Hz fs = 1000 # Frecuencia de muestreo duracion = 1.0 # segundos 

Generar señal 

t, senal_limpia, senal_ruidosa = generar_senal_compuesta(frecuencias, duracion, fs) 

2. Graficar señales antes del filtrado 

def graficar_senal(tiempo, senal_limpia, senal_ruidosa, titulo): plt.figure(figsize=(12, 6)) 

# Dominio del tiempo 
plt.subplot(2, 1, 1) 
plt.plot(tiempo, senal_limpia, label='Señal limpia') 
plt.plot(tiempo, senal_ruidosa, alpha=0.6, label='Señal con ruido') 
plt.title(f'{titulo} - Dominio del tiempo') 
plt.xlabel('Tiempo [s]') 
plt.ylabel('Amplitud') 
plt.legend() 
plt.grid(True) 
 
# Dominio de la frecuencia 
plt.subplot(2, 1, 2) 
n = len(senal_ruidosa) 
yf = fft(senal_ruidosa) 
xf = fftfreq(n, 1/fs)[:n//2] 
plt.plot(xf, 2/n * np.abs(yf[0:n//2])) 
plt.title(f'{titulo} - Dominio de la frecuencia') 
plt.xlabel('Frecuencia [Hz]') 
plt.ylabel('Magnitud') 
plt.grid(True) 
 
plt.tight_layout() 
plt.show() 
  

graficar_senal(t, senal_limpia, senal_ruidosa, 'Señal original') 

3. Diseñar filtros 

def disenar_filtro(filtro_tipo, orden, frec_corte, fs, frec_banda=None): """ Diseña un filtro digital 

Args: 
    filtro_tipo (str): 'lowpass', 'highpass' o 'bandpass' 
    orden (int): Orden del filtro 
    frec_corte (float or list): Frecuencia de corte (Hz) 
    fs (int): Frecuencia de muestreo 
    frec_banda (tuple): Solo para bandpass (frec_inf, frec_sup) 
     
Returns: 
    tuple: (b, a) coeficientes del filtro 
""" 
nyquist = 0.5 * fs 
 
if filtro_tipo == 'lowpass': 
    normal_cutoff = frec_corte / nyquist 
    b, a = signal.butter(orden, normal_cutoff, btype='low', analog=False) 
elif filtro_tipo == 'highpass': 
    normal_cutoff = frec_corte / nyquist 
    b, a = signal.butter(orden, normal_cutoff, btype='high', analog=False) 
elif filtro_tipo == 'bandpass': 
    low = frec_banda[0] / nyquist 
    high = frec_banda[1] / nyquist 
    b, a = signal.butter(orden, [low, high], btype='band', analog=False) 
else: 
    raise ValueError("Tipo de filtro no válido. Usar 'lowpass', 'highpass' o 'bandpass'") 
 
return b, a 
  

Diseñar filtros 

orden = 5 

Filtro pasa bajos (elimina frecuencias > 60 Hz) 

b_low, a_low = disenar_filtro('lowpass', orden, 60, fs) 

Filtro pasa altos (elimina frecuencias < 80 Hz) 

b_high, a_high = disenar_filtro('highpass', orden, 80, fs) 

Filtro pasa banda (solo deja entre 70-130 Hz) 

b_band, a_band = disenar_filtro('bandpass', orden, None, fs, frec_banda=(70, 130)) 

4. Aplicar filtros 

senal_filtrada_low = signal.filtfilt(b_low, a_low, senal_ruidosa) senal_filtrada_high = signal.filtfilt(b_high, a_high, senal_ruidosa) senal_filtrada_band = signal.filtfilt(b_band, a_band, senal_ruidosa) 

5. Visualizar resultados 

def graficar_resultados(t, original, filtrada, titulo, tipo_filtro): plt.figure(figsize=(12, 8)) 

# Dominio del tiempo 
plt.subplot(3, 1, 1) 
plt.plot(t, original, label='Original con ruido', alpha=0.7) 
plt.plot(t, filtrada, label=f'Filtrada ({tipo_filtro})', linewidth=2) 
plt.title(f'{titulo} - Dominio del tiempo') 
plt.xlabel('Tiempo [s]') 
plt.ylabel('Amplitud') 
plt.legend() 
plt.grid(True) 
 
# Dominio de la frecuencia (original) 
plt.subplot(3, 1, 2) 
n = len(original) 
yf = fft(original) 
xf = fftfreq(n, 1/fs)[:n//2] 
plt.plot(xf, 2/n * np.abs(yf[0:n//2])) 
plt.title('Espectro de frecuencia - Original') 
plt.xlabel('Frecuencia [Hz]') 
plt.ylabel('Magnitud') 
plt.grid(True) 
 
# Dominio de la frecuencia (filtrada) 
plt.subplot(3, 1, 3) 
yf_filt = fft(filtrada) 
plt.plot(xf, 2/n * np.abs(yf_filt[0:n//2])) 
plt.title(f'Espectro de frecuencia - Filtrada ({tipo_filtro})') 
plt.xlabel('Frecuencia [Hz]') 
plt.ylabel('Magnitud') 
plt.grid(True) 
 
plt.tight_layout() 
plt.show() 
  

Graficar para cada filtro 

graficar_resultados(t, senal_ruidosa, senal_filtrada_low, 'Filtro Pasa Bajos', 'Lowpass 60Hz') graficar_resultados(t, senal_ruidosa, senal_filtrada_high, 'Filtro Pasa Altos', 'Highpass 80Hz') graficar_resultados(t, senal_ruidosa, senal_filtrada_band, 'Filtro Pasa Banda', 'Bandpass 70-130Hz') 

6. Visualizar respuesta en frecuencia de los filtros 

def graficar_respuesta_frecuencia(b, a, fs, tipo): w, h = signal.freqz(b, a, worN=8000) frec = (w / np.pi) * (fs / 2) # Convertir a Hz 

plt.figure(figsize=(10, 5)) 
plt.plot(frec, 20 * np.log10(abs(h)), linewidth=2) 
plt.title(f'Respuesta en frecuencia - Filtro {tipo}') 
plt.xlabel('Frecuencia [Hz]') 
plt.ylabel('Amplitud [dB]') 
plt.grid(True) 
plt.ylim(-60, 5) 
plt.xlim(0, fs/2) 
plt.show() 
  

Graficar respuesta en frecuencia para cada filtro 

graficar_respuesta_frecuencia(b_low, a_low, fs, 'Pasa Bajos (60Hz)') graficar_respuesta_frecuencia(b_high, a_high, fs, 'Pasa Altos (80Hz)') graficar_respuesta_frecuencia(b_band, a_band, fs, 'Pasa Banda (70-130Hz)') 
