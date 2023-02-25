import RPi.GPIO as GPIO
import requests
import tkinter as tk
import tkinter.font
from tkinter import *
from tkinter import ttk
import time
import sys
from PIL import Image, ImageTk

EMULATE_HX711 = False

codigos_filhotes = requests.get('http://www.aveslegais.com/sistema/api_pesagem/novoFilhote.php').json()

win = tk.Tk()
win.geometry('800x800')

referenceUnit = 1009

if not EMULATE_HX711:
    import RPi.GPIO as GPIO
    from hx711 import HX711
else:
    from emulated_hx711 import HX711
hx = HX711(5, 6)


GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

def gravarAnilha():
    myObj = {'rfid_code': entry.get(), 'codigo': register_name.get()}
    x = requests.post('http://www.aveslegais.com/sistema/api_pesagem/salvaAnilhaProvisoria.php',json = myObj)
    print(x.json())
    
def clearEntry():
    entry.delete(0,END)
    entry.insert(0, "")

def pesagem():
    mensagemPesoGravado.pack_forget()
    entryCodigo['state']='normal'
    entryCodigo.delete(0,END)
    entryCodigo.insert(0, "")
    hx.set_reading_format("MSB", "MSB")
    hx.set_reference_unit(referenceUnit)
    win.update_idletasks()
    hx.reset()

    hx.tare()

    print ('Pronto. Adicione o animal. Espere a identificação para salvar')
    count = 0;
    while True:
        try:

        # These three lines are usefull to debug wether to use MSB or LSB in the reading formats
        # for the first parameter of "hx.set_reading_format("LSB", "MSB")".
        # Comment the two lines "val = hx.get_weight(5)" and "print val" and uncomment these three lines to see what it prints.

         #np_arr8_string = hx.get_np_arr8_string()
         #binary_string = hx.get_binary_string()
         #print (binary_string + " " + np_arr8_string)

        # Prints the weight. Comment if you're debbuging the MSB and LSB issue.

         val = max(0,round(hx.get_weight(5), 1))
         entryCodigo.focus()
         label_output.configure(text=val)
         win.update_idletasks()
         win.update()
         hx.power_down()
         hx.power_up()
         if(entryCodigo.get()):
             pb.pack()
             pb.start(10)
             entryCodigo['state']='disabled'
             count= count + 1
             if(count==10):
                 pb.stop()
                 pb.pack_forget()
                 mensagemPesoGravado.pack()
                 peso = {'code':entryCodigo.get(), 'peso': label_output['text']}
                 x = requests.post('http://www.aveslegais.com/sistema/api_pesagem/nova_pesagem.php',json=peso)
                 print(x.json())
                 break
             
         time.sleep(1.0)
        except (KeyboardInterrupt, SystemExit):

            cleanAndExit()


win.title('pesagem automatica')

imagem = ImageTk.PhotoImage(Image.open('logo-comfauna.jpeg').resize((125, 50)))
w = Label(win,image=imagem)
w.grid(column=0,row=0)   


tabControl = ttk.Notebook(win, width=800, height=800)
tab1 = ttk.Frame(tabControl);
tab2 = ttk.Frame(tabControl);
tabControl.add(tab1,text='Nova Pesagem')
tabControl.add(tab2,text='Novo Cadastro')
tabControl.grid(row=0,column=1)
myFontNormal = tkinter.font.Font(family='Helvetica', size=12)
myFont = tkinter.font.Font(family='Helvetica', size=12, weight='bold')
myFontBig = tkinter.font.Font(family='Helvetica', size=20, weight='bold')
myFontWarning = tkinter.font.Font(family='Helvetica', size=17)
ledButton = tk.Button(
    tab1,
    text='Tara',
    font=myFont,
    command=pesagem,
    bg='aliceblue',
    height=2,
    width=30,
    )


ledButton.grid(row=50,column=2,pady=30)
#sv = StringVar()
#sv.trace("w", lambda name, index, mode, sv=sv: print('oi'))
#entryCodigo= Entry(tab1,textvariable=sv)
entryCodigo= Entry(tab1)
entryCodigo.grid(row=70, column=2, pady=(10,40))
label_output = tk.Label(tab1, text='00.00',font=myFontBig, height=2,width=30)
label_output.grid(row=60,column=2)

frame = ttk.Frame(tab1)
pb = ttk.Progressbar(frame, length=100, mode='indeterminate')
frame.grid(row=90, column=2)
mensagemPesoGravado = Message(frame, width = 400, text='Peso Gravado!', padx=30, pady=10, font=myFont, bg='#9FE2BF')


label_register_name = tk.Label(tab2, font=myFontNormal, text="Código do Filhote *")
register_name = ttk.Combobox(tab2,
    font=myFont
    )
register_name.bind(gravarAnilha)
entry= Entry(tab2)
entry.place(width=15,height=50)
entry.grid(row=1, column=2, pady=10)

button_register = tk.Button(tab2,
    text='Gravar Anilha',
    font=myFont,
    command=gravarAnilha,
    bg='aliceblue',
    height=2,
    width=20,)
button_clear = tk.Button(tab2,
    text='Limpar Gravação',
    font=myFont,
    command= clearEntry,
    bg='#fbe299',
    height=2,
    width=15,)
register_name['values'] = codigos_filhotes

label_register_name.grid(row=0,column=1, pady=5, padx=5)
register_name.grid(row=1, column=1, padx=20)

button_register.grid(row=3, column=1, pady=5,padx=5)
button_clear.grid(row=3, column=2, pady=5,padx=5)

tk.mainloop()

