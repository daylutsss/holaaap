#Entradas
import tkinter as tk
from tkinter import ttk, messagebox
from datetime import date, timedelta

#Define el tipo de letra, tamaño y color que se usara 
FUENTE = ("Times New Roman", 10)
COLOR_FONDO_APP = "SteelBlue2"      
COLOR_ENTRADA_FONDO = "SteelBlue4"
COLOR_ENTRADA_TEXTO = "white"

#Es la base que nos dice que debemos poner y sacar dinero
class Transacciones:
    """Interfaz simple para abonar / cargar saldo"""
    def abonar(self, cantidad):
        raise NotImplementedError

    def cargar(self, cantidad):
        raise NotImplementedError

#En si, guarda los datos del cliente
class Cliente:
    def __init__(self, nombre, apellido_paterno, apellido_materno, fecha_nacimiento, domicilio):
        self.nombre = nombre
        self.apellido_paterno = apellido_paterno
        self.apellido_materno = apellido_materno
        self.fecha_nacimiento = fecha_nacimiento
        self.domicilio = domicilio

#Guarda el número de cuenta y el dinero, empieza con $1000, tiene la función de poner
#y sacar el dinero y mo deja sacar dinero si no hay
class Cuenta(Transacciones):
    def __init__(self, numero_cuenta):
        self.numero_cuenta = numero_cuenta
        self.saldo = 1000.0

    def abonar(self, cantidad):
        self.saldo += cantidad

    def cargar(self, cantidad):
        if self.saldo >= cantidad:
            self.saldo -= cantidad
            return True
        else:
            return False

#Guarda la información de si se mete o saca dinero y guarda la fecha
#También cuanto dinero se metió, se sacó y cuanto quedó
class Movimiento:
    def __init__(self, fecha_movimiento, descripcion, cargo, abono, saldo):
        self.fecha_movimiento = fecha_movimiento
        self.descripcion = descripcion
        self.cargo = cargo
        self.abono = abono
        self.saldo = saldo

#Guarda al cliente, la cuenta y todo lo que se han hecho, también se pueden hacer cargos o
#abonos y revisar si se puede hacer
class EstadoDeCuenta:
    def __init__(self, cliente: Cliente, cuenta: Cuenta, fecha_ingreso: date):
        self.cliente = cliente
        self.cuenta = cuenta
        self.fecha_ingreso = fecha_ingreso
        self.movimientos = []

    def agregar_movimiento(self, descripcion, tipo, cantidad):
        """Agrega un movimiento; tipo 'Abono' o 'Cargo'. Retorna False si no se pudo hacer por saldo insuficiente."""
        fecha = self.fecha_ingreso + timedelta(days=len(self.movimientos))
        cargo = 0.0
        abono = 0.0
        if tipo == "Abono":
            self.cuenta.abonar(cantidad)
            abono = cantidad
        elif tipo == "Cargo":
            if self.cuenta.cargar(cantidad):
                cargo = cantidad
            else:
                return False
        else:
            return False

        saldo_actual = self.cuenta.saldo
        mov = Movimiento(fecha, descripcion, cargo, abono, saldo_actual)
        self.movimientos.append(mov)
        return True

    def obtener_totales(self):
        total_cargos = sum(m.cargo for m in self.movimientos)
        total_abonos = sum(m.abono for m in self.movimientos)
        saldo_final = self.cuenta.saldo
        return total_cargos, total_abonos, saldo_final

    def generar_texto_estado(self):
        """Genera todo el estado de cuenta como un texto para mostrar."""
        lines = []
        lines.append("===== ESTADO DE CUENTA =====\n")
        lines.append(">> Cliente:")
        lines.append(f"Nombre: {self.cliente.nombre} {self.cliente.apellido_paterno} {self.cliente.apellido_materno}")
        lines.append(f"Fecha de nacimiento: {self.cliente.fecha_nacimiento}")
        lines.append(f"Domicilio: {self.cliente.domicilio}\n")

        lines.append(">> Cuenta:")
        lines.append(f"Número de cuenta: {self.cuenta.numero_cuenta}")
        lines.append(f"Saldo inicial: $1000.00\n")

        lines.append(">> Movimientos:")
        lines.append(f"{'Fecha':10} | {'Descripción':20} | {'Cargo':>10} | {'Abono':>10} | {'Saldo':>10}")
        lines.append("-" * 70)
        for mov in self.movimientos:
            lines.append(f"{mov.fecha_movimiento} | {mov.descripcion[:20]:20} | {mov.cargo:10.2f} | {mov.abono:10.2f} | {mov.saldo:10.2f}")

        total_cargos, total_abonos, saldo_final = self.obtener_totales()
        lines.append("\n>> Totales:")
        lines.append(f"Total cargos: ${total_cargos:.2f}")
        lines.append(f"Total abonos: ${total_abonos:.2f}")
        lines.append(f"Saldo final: ${saldo_final:.2f}")
        lines.append("=============================")
        return "\n".join(lines)

#Aquí se hace la ventana, los botones y las cajas para escribir, se acomoda primero lo de
#los datos del cliente y la cuenta, después se agrega lo de la descripción, si quieres hacer
#cargo o abono con Combobox y de cuanto, y al final da el resumen de los datos y todo eso
class EdoCuenta:
    def __init__(self, root):
        self.root = root
        self.root.title("Estado de Cuenta Bancario")
        self.estado = None
        self.movimientos_hechos = 0

        self.root.configure(bg=COLOR_FONDO_APP)

        self._crear_widgets_cliente()
        self._crear_widgets_movimiento()
        self._crear_widget_resumen()

    def _crear_widgets_cliente(self):
        frame = tk.LabelFrame(self.root, text="Datos del Cliente / Cuenta", bg=COLOR_FONDO_APP, font=FUENTE, fg="black")
        frame.grid(row=0, column=0, padx=10, pady=10, sticky="ew")

        self.var_nombre = tk.StringVar()
        self.var_apellido_p = tk.StringVar()
        self.var_apellido_m = tk.StringVar()
        self.var_fecha_nac = tk.StringVar()
        self.var_domicilio = tk.StringVar()
        self.var_num_cuenta = tk.StringVar()

        campos = [
            ("Nombre:", self.var_nombre),
            ("Apellido Paterno:", self.var_apellido_p),
            ("Apellido Materno:", self.var_apellido_m),
            ("Fecha Nacimiento (YYYY-MM-DD):", self.var_fecha_nac),
            ("Domicilio:", self.var_domicilio),
            ("Número de cuenta:", self.var_num_cuenta),
        ]

        for i, (label_text, var) in enumerate(campos):
            tk.Label(frame, text=label_text, font=FUENTE, bg=COLOR_FONDO_APP).grid(row=i, column=0, sticky="w")
            tk.Entry(frame, textvariable=var, font=FUENTE, bg=COLOR_ENTRADA_FONDO, fg=COLOR_ENTRADA_TEXTO).grid(row=i, column=1, sticky="ew")

        boton = tk.Button(frame, text="Crear Cuenta", font=FUENTE, bg="white", command=self.crear_cuenta)
        boton.grid(row=len(campos), column=1, pady=5, sticky="e")

    def _crear_widgets_movimiento(self):
        frame = tk.LabelFrame(self.root, text="Agregar Movimiento", bg=COLOR_FONDO_APP, font=FUENTE, fg="black")
        frame.grid(row=1, column=0, padx=10, pady=10, sticky="ew")

        self.var_descripcion = tk.StringVar()
        self.var_tipo = tk.StringVar(value="Abono")
        self.var_monto = tk.StringVar()

        tk.Label(frame, text="Descripción:", font=FUENTE, bg=COLOR_FONDO_APP).grid(row=0, column=0, sticky="w")
        tk.Entry(frame, textvariable=self.var_descripcion, font=FUENTE, bg=COLOR_ENTRADA_FONDO, fg=COLOR_ENTRADA_TEXTO).grid(row=0, column=1, sticky="ew")

        tk.Label(frame, text="Tipo:", font=FUENTE, bg=COLOR_FONDO_APP).grid(row=1, column=0, sticky="w")
        combo = ttk.Combobox(frame, textvariable=self.var_tipo, values=("Abono", "Cargo"), state="readonly", font=FUENTE)
        combo.grid(row=1, column=1, sticky="ew")

        tk.Label(frame, text="Monto:", font=FUENTE, bg=COLOR_FONDO_APP).grid(row=2, column=0, sticky="w")
        tk.Entry(frame, textvariable=self.var_monto, font=FUENTE, bg=COLOR_ENTRADA_FONDO, fg=COLOR_ENTRADA_TEXTO).grid(row=2, column=1, sticky="ew")

        boton = tk.Button(frame, text="Agregar movimiento", font=FUENTE, bg="white", command=self.agregar_movimiento)
        boton.grid(row=3, column=1, pady=5, sticky="e")

    def _crear_widget_resumen(self):
        frame = tk.LabelFrame(self.root, text="Estado de Cuenta - Resumen", bg=COLOR_FONDO_APP, font=FUENTE, fg="black")
        frame.grid(row=2, column=0, padx=10, pady=10, sticky="nsew")

        self.text_resumen = tk.Text(frame, width=80, height=20, font=FUENTE, bg=COLOR_ENTRADA_FONDO, fg=COLOR_ENTRADA_TEXTO)
        self.text_resumen.pack(fill="both", expand=True)

        boton = tk.Button(frame, text="Mostrar Estado de Cuenta", font=FUENTE, bg="white", command=self.mostrar_estado)
        boton.place(x=1100, y=250)
    #Cuando están los datos del cliente y la cuenta, le picas en Crear Cuenta, se crea todo
    #y si algo falta te dice
    def crear_cuenta(self):
        if not (self.var_nombre.get() and self.var_apellido_p.get() and self.var_apellido_m.get() and self.var_fecha_nac.get() and self.var_domicilio.get() and self.var_num_cuenta.get()):
            messagebox.showwarning("Datos faltantes", "Por favor completa todos los datos del cliente y la cuenta.")
            return

        try:
            cliente = Cliente(
                self.var_nombre.get(),
                self.var_apellido_p.get(),
                self.var_apellido_m.get(),
                self.var_fecha_nac.get(),
                self.var_domicilio.get()
            )
            cuenta = Cuenta(self.var_num_cuenta.get())
            self.estado = EstadoDeCuenta(cliente, cuenta, date.today())
            messagebox.showinfo("Cuenta creada", "La cuenta ha sido creada con saldo inicial de $1000.00")
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo crear la cuenta: {e}")
    #Checa que haya una cuenta, y que todo sea válido, te muestra un mensaje si esta mal,
    #o limpia si esta bien
    def agregar_movimiento(self):
        if self.estado is None:
            messagebox.showwarning("Cuenta no creada", "Primero debes crear la cuenta.")
            return

        descripcion = self.var_descripcion.get()
        tipo = self.var_tipo.get()
        monto_text = self.var_monto.get()

        if not descripcion or not monto_text:
            messagebox.showwarning("Datos faltantes", "Por favor escribe descripción y monto.")
            return

        try:
            monto = float(monto_text)
            if monto <= 0:
                messagebox.showwarning("Monto inválido", "El monto debe ser mayor que 0.")
                return
        except ValueError:
            messagebox.showwarning("Monto inválido", "Introduce un número válido para monto.")
            return

        ok = self.estado.agregar_movimiento(descripcion, tipo, monto)
        if not ok:
            messagebox.showwarning("Saldo insuficiente", "No se puede hacer el cargo porque no hay saldo suficiente.")
        else:
            self.movimientos_hechos += 1
            self.var_descripcion.set("")
            self.var_monto.set("")
    #Muestra la lista con todos los movimientos y el dinero que queda
    def mostrar_estado(self):
        if self.estado is None:
            messagebox.showwarning("Cuenta no creada", "Primero debes crear la cuenta.")
            return

        texto = self.estado.generar_texto_estado()
        self.text_resumen.delete("1.0", tk.END)
        self.text_resumen.insert(tk.END, texto)

#Aquí se abre la ventana y empieza todo para usar el programa
if __name__ == "__main__":
    root = tk.Tk()
    root.columnconfigure(0, weight=1)
    root.rowconfigure(2, weight=1)
    app = EdoCuenta(root)
    root.mainloop()
    
