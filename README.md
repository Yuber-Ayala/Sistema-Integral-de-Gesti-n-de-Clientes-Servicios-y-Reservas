# Sistema-Integral-de-Gesti-n-de-Clientes-Servicios-y-Reservas
Implementar el manejo de excepciones en el desarrollo de aplicaciones orientadas a objetos buscando estabilidad y robustez, permitiendo una gestión adecuada de errores en el funcionamiento de las soluciones


import abc      #(Abstract Base Classes) Sirve para crear una plantilla base que no se puede usar directamente,sino que obliga 
                #a las clases hijas a heredar de ella e implementar obligatoriamente ciertos métodos.
import datetime     #Permite obtener la fecha y hora exacta del sistema en el momento en que ocurre un evento.
                    #para saber con precisión en qué segundo se registró cada evento o error dentro del archivo


# 1. MANEJO DE LOGS Y EXCEPCIONES PERSONALIZADAS


def registrar_log(mensaje: str, es_error: bool = False):
    """
    Función auxiliar para registrar eventos y errores en un archivo físico de texto.
    Usa el bloque try/except/finally para garantizar el cierre correcto del archivo.
    """
    fecha_hora = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    tipo = "ERROR" if es_error else "INFO"
    linea = f"[{fecha_hora}] [{tipo}] {mensaje}\n"
    
    archivo = None
    try:
        archivo = open("sistema_errores.log", "a", encoding="utf-8-sig")
        archivo.write(linea)
    except IOError as e:
        print(f"Error crítico al intentar escribir en el archivo de logs: {e}")
    finally:
        if archivo:
            archivo.close()


class SoftwareFJError(Exception):
    """Excepción base personalizada para el sistema Software FJ."""
    pass


class DatosInvalidosError(SoftwareFJError):
    """Excepción lanzada cuando los datos de entrada no cumplen las validaciones."""
    pass


class ReservaInvalidaError(SoftwareFJError):
    """Excepción lanzada cuando ocurre un problema en la gestión de reservas."""
    pass



# 2. ARQUITECTURA DE CLASES (POO Y ABSTRACCIÓN)


class EntidadBase(abc.ABC):
    """
    Clase abstracta que sirve de molde base para cualquier entidad del sistema.
    Aplica el principio de Abstracción.
    """
    def __init__(self, id_entidad: str):
        if not id_entidad or not isinstance(id_entidad, str):
            raise DatosInvalidosError("El ID de la entidad debe ser una cadena no vacía.")
        self._id_entidad = id_entidad

    @property
    def id_entidad(self) -> str:
        return self._id_entidad

    @abc.abstractmethod
    def obtener_resumen(self) -> str:
        """Método abstracto que obliga a las subclases a definir su resumen."""
        pass


class Cliente(EntidadBase):
    """
    Clase que representa un cliente.
    Aplica Encapsulación utilizando atributos privados (_nombre, _correo, _telefono) y getters.
    """
    def __init__(self, id_cliente: str, nombre: str, correo: str, telefono: str):
        super().__init__(id_cliente)
        self.nombre = nombre
        self.correo = correo
        self.telefono = telefono

    # Getter y Setter con validación para Nombre
    @property
    def nombre(self) -> str:
        return self._nombre

    @nombre.setter
    def nombre(self, valor: str):
        if not valor or len(valor.strip()) < 3:
            raise DatosInvalidosError(f"Nombre inválido: '{valor}'. Debe tener al menos 3 caracteres.")
        self._nombre = valor.strip()

    # Getter y Setter con validación para Correo
    @property
    def correo(self) -> str:
        return self._correo

    @correo.setter
    def correo(self, valor: str):
        if "@" not in valor or "." not in valor:
            raise DatosInvalidosError(f"Correo electrónico inválido: '{valor}'.")
        self._correo = valor.strip()

    # Getter y Setter para Teléfono
    @property
    def telefono(self) -> str:
        return self._telefono

    @telefono.setter
    def telefono(self, valor: str):
        if not valor.isdigit() or len(valor) < 7:
            raise DatosInvalidosError(f"Teléfono inválido: '{valor}'. Debe contener solo números y al menos 7 dígitos.")
        self._telefono = valor

    def obtener_resumen(self) -> str:
        return f"Cliente ID: {self.id_entidad} | Nombre: {self.nombre} | Correo: {self.correo}"


class Servicio(EntidadBase):
    """
    Clase abstracta Servicio que hereda de EntidadBase.
    Base para aplicar Polimorfismo en el cálculo de costos.
    """
    def __init__(self, id_servicio: str, nombre_servicio: str, costo_base: float):
        super().__init__(id_servicio)
        if costo_base <= 0:
            raise DatosInvalidosError(f"El costo base debe ser mayor a cero. Valor ingresado: {costo_base}")
        self._nombre_servicio = nombre_servicio
        self._costo_base = costo_base

    @property
    def nombre_servicio(self) -> str:
        return self._nombre_servicio

    @property
    def costo_base(self) -> float:
        return self._costo_base

    @abc.abstractmethod
    def calcular_costo(self, duracion_horas: float, descuento: float = 0.0, impuesto: float = 0.19) -> float:
        """Método abstracto para el cálculo de costos con polimorfismo y sobrecarga simulada."""
        pass
        # ==========================================
# MODULO DE GESTION DE RESERVAS
# Proyecto: Software FJ
# Aporte: Gestion de reservas
# ==========================================

class ReservaInvalidaError(Exception):
    """Error personalizado para reservas incorrectas."""
    pass


class Reserva:
    """Clase encargada de gestionar las reservas."""

    def __init__(self, id_reserva, cliente, servicio, duracion_horas):

        if not id_reserva:
            raise ReservaInvalidaError("El ID de reserva es obligatorio.")

        if cliente is None or servicio is None:
            raise ReservaInvalidaError("Debe existir un cliente y un servicio.")

        if duracion_horas <= 0:
            raise ReservaInvalidaError("La duración debe ser mayor que cero.")

        self.__id_reserva = id_reserva
        self.__cliente = cliente
        self.__servicio = servicio
        self.__duracion_horas = duracion_horas
        self.__estado = "Pendiente"

    @property
    def estado(self):
        return self.__estado

    def confirmar(self):
        if self.__estado != "Pendiente":
            raise ReservaInvalidaError("La reserva no se puede confirmar.")

        self.__estado = "Confirmada"
        return "Reserva confirmada correctamente."

    def cancelar(self):
        if self.__estado == "Cancelada":
            raise ReservaInvalidaError("La reserva ya está cancelada.")

        self.__estado = "Cancelada"
        return "Reserva cancelada correctamente."



# ==============================================================================
# 5. SIMULACIÓN DE 10 OPERACIONES COMPLETAS (PRUEBA DE ESTABILIDAD)
# ==============================================================================

if __name__ == "__main__":
    print("=================================================================")
    print("   SISTEMA DE GESTIÓN SOFTWARE FJ - SIMULACIÓN DE OPERACIONES   ")
    print("=================================================================\n")

    # Listas internas para almacenamiento en memoria (sin base de datos)
    clientes_registrados = []
    servicios_registrados = []
    reservas_registradas = []

    # Operación 1: Registro exitoso de Cliente 1
    print("--- Operación 1: Registro correcto de Cliente 1 ---")
    try:
        c1 = Cliente("C01", "Yuber Ayala", "yuber@correo.com", "3101234567")
        clientes_registrados.append(c1)
        registrar_log(f"Cliente registrado: {c1.obtener_resumen()}")
        print(f"Éxito: {c1.obtener_resumen()}")
    except SoftwareFJError as e:
        registrar_log(f"Error en Op 1: {e}", es_error=True)
        print(f"Error capturado: {e}")

    # Operación 2: Intento de registro de Cliente con correo inválido (Fallo controlado)
    print("\n--- Operación 2: Registro de Cliente con correo erróneo ---")
    try:
        c2 = Cliente("C02", "Pedro Gómez", "correo_sin_arroba_com", "3009876543")
        clientes_registrados.append(c2)
    except SoftwareFJError as e:
        registrar_log(f"Error en Op 2 (Captura esperada): {e}", es_error=True)
        print(f"Excepción capturada con éxito: {e}")

    # Operación 3: Creación exitosa de Servicio 1 (Reserva de Sala)
    print("\n--- Operación 3: Registro de Servicio (Reserva de Sala) ---")
    try:
        s1 = ReservaSala("S01", "Sala VIP Ejecutivo", costo_base=50.0, capacidad_personas=25)
        servicios_registrados.append(s1)
        registrar_log(f"Servicio registrado: {s1.obtener_resumen()}")
        print(f"Éxito: {s1.obtener_resumen()}")
    except SoftwareFJError as e:
        registrar_log(f"Error en Op 3: {e}", es_error=True)
        print(f"Error capturado: {e}")

    # Operación 4: Creación errónea de Servicio con costo negativo (Fallo controlado)
    print("\n--- Operación 4: Registro de Servicio con costo negativo ---")
    try:
        s_invalid = AlquilerEquipo("S02", "Proyector 4K", costo_base=-15.0)
    except SoftwareFJError as e:
        registrar_log(f"Error en Op 4 (Captura esperada): {e}", es_error=True)
        print(f"Excepción capturada con éxito: {e}")

    # Operación 5: Creación exitosa de Servicio 2 (Alquiler de Equipo) y Servicio 3 (Asesoría)
    print("\n--- Operación 5: Registro de Servicio de Alquiler de Equipo y Asesoría ---")
    try:
        s2 = AlquilerEquipo("S02", "Servidor Portátil de Pruebas", costo_base=30.0, requiere_deposito=True)
        s3 = AsesoriaEspecializada("S03", "Consultoría en Arquitectura Cloud", costo_base=80.0, nivel_consultor="SENIOR")
        servicios_registrados.extend([s2, s3])
        print(f"Éxito: {s2.obtener_resumen()}")
        print(f"Éxito: {s3.obtener_resumen()}")
    except SoftwareFJError as e:
        registrar_log(f"Error en Op 5: {e}", es_error=True)
        print(f"Error capturado: {e}")


