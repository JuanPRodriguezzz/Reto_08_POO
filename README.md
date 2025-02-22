# Reto_08_POO
## Juan Pablo Rodríguez Cruz

### 1. The restaurant class revisted like for the fourth time.
The menu once again....(I am running out of ideas). Well the task is quite easy, take the Menu code from Reto 3 and implement a new Class that creates and iterable with all the items in an order, it should allow looping and contain all item attributes.

## Code
```python
import os
import json
from collections import namedtuple

Definimos una cola FIFO para manejar múltiples órdenes
class OrderManager:
    """Maneja múltiples órdenes en una estructura FIFO."""
    def __init__(self):
        # En vez de deque(), usamos una simple lista
        self._orders_queue = []

    def add_order(self, order):
        # Agregamos al final de la lista
        self._orders_queue.append(order)

    def get_next_order(self):
        # Obtenemos el primer elemento, si existe
        if self._orders_queue:
            return self._orders_queue.pop(0)
        return None

    def has_orders(self):
        return len(self._orders_queue) > 0


# Definimos un namedtuple para representar conjuntos de ítems (por ejemplo, un combo)
Combo = namedtuple("Combo", ["combo_name", "appetizer", "beverage", "main_course"])


class MenuItem:
    """Base class para un ítem del menú."""
    def __init__(self, name, price):
        self._name = name
        self._price = price

    def get_name(self):
        return self._name

    def set_name(self, name):
        self._name = name

    def get_price(self):
        return self._price

    def set_price(self, price):
        self._price = price

    def calculate_price(self):
        return self._price


class Beverage(MenuItem):
    """Subclass para bebidas."""
    def __init__(self, name, price, is_alcoholic):
        super().__init__(name, price)
        self._is_alcoholic = is_alcoholic

    def get_is_alcoholic(self):
        return self._is_alcoholic

    def set_is_alcoholic(self, value):
        self._is_alcoholic = value


class Appetizer(MenuItem):
    """Subclass para aperitivos."""
    pass


class MainCourse(MenuItem):
    """Subclass para platos principales."""
    def __init__(self, name, price, is_vegetarian):
        super().__init__(name, price)
        self._is_vegetarian = is_vegetarian

    def get_is_vegetarian(self):
        return self._is_vegetarian

    def set_is_vegetarian(self, value):
        self._is_vegetarian = value


# Nueva clase para crear un iterable con todos los ítems de una orden
class OrderItemsIterable:
    """
    Permite iterar sobre los ítems de la orden y ver todos sus atributos importantes.
    """
    def __init__(self, order):
        self._items = order.items
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self):
        # Retorna el mismo item o una tupla con los atributos
        if self._index < len(self._items):
            item = self._items[self._index]
            self._index += 1
            return {
                "class": item.__class__.__name__,
                "name": item.get_name(),
                "price": item.get_price()
            }
        else:
            raise StopIteration


class Order:
    """Clase que representa la orden de un cliente."""
    def __init__(self):
        self.items = []

    def add_item(self, item):
        if isinstance(item, MenuItem):
            self.items.append(item)
        else:
            raise ValueError("El ítem debe ser un objeto de tipo MenuItem.")

    def calculate_total(self):
        return sum(item.calculate_price() for item in self.items)

    def apply_discount(self, discount_percentage):
        total = self.calculate_total()
        discount = total * (discount_percentage / 100)
        return total - discount

    def calculate_total_price(self):
        total = self.calculate_total()

        # Descuento si existe un MainCourse y una Beverage
        has_main_course = any(isinstance(item, MainCourse) for item in self.items)
        has_beverage = any(isinstance(item, Beverage) for item in self.items)

        if has_main_course and has_beverage:
            total *= 0.9  # 10% discount

        return total

    def show_order_summary(self):
        print("Resumen del pedido:")
        for item in self.items:
            print(f"{item.get_name()}: ${item.get_price():.2f}")
        print(f"Total: ${self.calculate_total_price():.2f}")

    @staticmethod
    def limpiar_pantalla():
        """Limpia la consola."""
        os.system("cls" if os.name == "nt" else "clear")

    @staticmethod
    def continuar():
        """Pausa la ejecución hasta que el usuario presione Enter."""
        input("\nPresiona Enter para continuar...")
        Order.limpiar_pantalla()


class Payment:
    """Abstract base class para métodos de pago."""
    def pay(self, amount):
        raise NotImplementedError("Subclases deben implementar el método pay.")


class CreditCard(Payment):
    def __init__(self, card_number, cvv):
        self.card_number = card_number
        self.cvv = cvv

    def pay(self, amount):
        print(f"Pagando ${amount:.2f} con tarjeta terminada en {self.card_number[-4:]}")


class Cash(Payment):
    def __init__(self, amount_given):
        self.amount_given = amount_given

    def pay(self, amount):
        if self.amount_given >= amount:
            change = self.amount_given - amount
            print(f"Pago realizado en efectivo. Cambio: ${change:.2f}")
        else:
            shortage = amount - self.amount_given
            print(f"Fondos insuficientes. Faltan ${shortage:.2f} para completar el pago.")


# Clase para administrar el menú, guardándolo/leyéndolo como JSON
class MenuManager:
    """Permite agregar, actualizar y eliminar ítems del menú, almacenándolos en un archivo JSON."""
    def __init__(self, json_file="menu.json"):
        self.json_file = json_file
        self.menu_data = self.load_menu()

    def load_menu(self):
        if os.path.exists(self.json_file):
            with open(self.json_file, "r", encoding="utf-8") as f:
                return json.load(f)
        else:
            return {"Beverage": [], "Appetizer": [], "MainCourse": []}

    def save_menu(self):
        with open(self.json_file, "w", encoding="utf-8") as f:
            json.dump(self.menu_data, f, indent=4, ensure_ascii=False)

    def add_item(self, category, name, price, **kwargs):
        """
        category: "Beverage", "Appetizer", o "MainCourse"
        kwargs: argumentos extra como 'is_alcoholic' o 'is_vegetarian'
        """
        if category not in self.menu_data:
            self.menu_data[category] = []
        new_item = {"name": name, "price": price}
        new_item.update(kwargs)
        self.menu_data[category].append(new_item)
        self.save_menu()
        print(f"Item '{name}' agregado correctamente en {category}.")

    def update_item(self, category, old_name, new_name=None, new_price=None, **kwargs):
        if category not in self.menu_data:
            print("Categoría no encontrada.")
            return
        for item in self.menu_data[category]:
            if item["name"] == old_name:
                if new_name is not None:
                    item["name"] = new_name
                if new_price is not None:
                    item["price"] = new_price
                # Actualizar kwargs (ej. is_alcoholic, is_vegetarian)
                for k, v in kwargs.items():
                    item[k] = v
                self.save_menu()
                print(f"Item '{old_name}' actualizado correctamente.")
                return
        print(f"Item '{old_name}' no encontrado en la categoría {category}.")

    def delete_item(self, category, item_name):
        if category not in self.menu_data:
            print("Categoría no encontrada.")
            return
        initial_len = len(self.menu_data[category])
        self.menu_data[category] = [i for i in self.menu_data[category] if i["name"] != item_name]
        if len(self.menu_data[category]) < initial_len:
            self.save_menu()
            print(f"Item '{item_name}' eliminado de {category}.")
        else:
            print(f"No se encontró el item '{item_name}' en {category}.")


def build_menu_objects(menu_manager):
    """
    Construye objetos de tipo Beverage, Appetizer y MainCourse a partir del
    diccionario en memoria que maneja MenuManager. Retorna una lista de MenuItem.
    """
    menu_objects = []
    for item in menu_manager.menu_data.get("Beverage", []):
        b = Beverage(
            item["name"], 
            float(item["price"]), 
            item.get("is_alcoholic", False)
        )
        menu_objects.append(b)
    for item in menu_manager.menu_data.get("Appetizer", []):
        a = Appetizer(item["name"], float(item["price"]))
        menu_objects.append(a)
    for item in menu_manager.menu_data.get("MainCourse", []):
        m = MainCourse(
            item["name"], 
            float(item["price"]), 
            item.get("is_vegetarian", False)
        )
        menu_objects.append(m)
    return menu_objects


def main_menu():
    # Cargar o crear el menu desde el JSON
    menu_manager = MenuManager("menu.json")
    menu = build_menu_objects(menu_manager)

    order = Order()

    while True:
        Order.limpiar_pantalla()
        print("Menú del Restaurante:")
        print("\n--- Bebidas ---")
        bev_indices = []
        idx = 1
        for item in menu:
            if isinstance(item, Beverage):
                print(f"{idx}. {item.get_name()} - ${item.get_price():.2f}")
                bev_indices.append(idx)
            idx += 1

        print("\n--- Aperitivos ---")
        app_indices = []
        for item in menu:
            if isinstance(item, Appetizer):
                print(f"{idx}. {item.get_name()} - ${item.get_price():.2f}")
                app_indices.append(idx)
            idx += 1

        print("\n--- Platos principales ---")
        main_indices = []
        for item in menu:
            if isinstance(item, MainCourse):
                print(f"{idx}. {item.get_name()} - ${item.get_price():.2f}")
                main_indices.append(idx)
            idx += 1

        try:
            choices = input("\nIngrese los números de los items separados por comas: ").strip()
            choices = [int(choice) for choice in choices.split(",") if choice.strip().isdigit()]

            for choice in choices:
                # El choice es 1-based, necesitamos 0-based
                index_in_list = choice - 1
                if 0 <= index_in_list < len(menu):
                    order.add_item(menu[index_in_list])
                    print(f"{menu[index_in_list].get_name()} agregado al pedido.")
                else:
                    print(f"El número {choice} no es válido y se ignoró.")
        except ValueError:
            print("\nEntrada inválida. Por favor, ingrese solo números separados por comas.")

        confirm = input("\n¿Desea agregar más items? (s/n): ").strip().lower()
        if confirm != 's':
            break

    Order.limpiar_pantalla()
    print("\nResumen Final del Pedido:")
    order.show_order_summary()

    # Ejemplo de uso de la clase OrderItemsIterable
    print("\nRecorriendo la orden:")
    order_iterable = OrderItemsIterable(order)
    for item_dict in order_iterable:
        print(f"- {item_dict}")

    confirm = input("\n¿Confirma su pedido? (s/n): ").strip().lower()
    if confirm == 's':
        payment_method = input("\nSeleccione método de pago (1: Tarjeta, 2: Efectivo): ").strip()
        if payment_method == '1':
            card_number = input("Ingrese el número de tarjeta: ").strip()
            cvv = input("Ingrese el CVV: ").strip()
            card = CreditCard(card_number, cvv)
            card.pay(order.calculate_total_price())
        elif payment_method == '2':
            try:
                amount_given = float(input("Ingrese el monto entregado: ").strip())
                cash = Cash(amount_given)
                cash.pay(order.calculate_total_price())
            except ValueError:
                print("\nMonto ingresado no válido.")
        else:
            print("\nMétodo de pago no válido")
    else:
        print("\nEl pedido fue cancelado.")


if __name__ == "__main__":
    # Ejemplo de uso con varias ordenes:
    order_manager = OrderManager()
    # order1 = Order()
    # order2 = Order()
    # order_manager.add_order(order1)
    # order_manager.add_order(order2)

    main_menu()
```
