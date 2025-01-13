import tkinter as tk
import socket
import threading
import re


BROADCAST_IP = "0.0.0.0"
UDP_PORT = 5005
global_sock = None

def create_global_socket():
    global global_sock
    global_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    global_sock.bind((BROADCAST_IP, UDP_PORT))

def is_valid_ip(ip):
    """
    Funkcja sprawdzająca poprawność adresu IP.
    """
    octets = ip.split('.')
    if len(octets) != 4:
        return False
    for octet in octets:
        if not octet.isdigit() or int(octet) < 0 or int(octet) > 255:
            return False
    return True

def on_button_click(message, robot_udp_ip):
    print("Przycisk został kliknięty!")
    # Tworzenie gniazda UDP
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    # Wysłanie wiadomości po UDP
    sock.sendto(message.encode(), (robot_udp_ip, UDP_PORT))  # Konwertowanie stringa na bajty i wysłanie
    # Zamykanie gniazda
    sock.close()

def receive_udp_packets(robot_udp_ip, position_text, kp_text, text_box, alias):
    global global_sock

    while True:
        try:
            data, addr = global_sock.recvfrom(1024)  # Odbieranie danych
            decoded_data = data.decode()

            # Check if the message starts with the exact alias followed by a space
            if alias and decoded_data.startswith(alias + " "):
                # Remove the alias from the message
                stripped_data = decoded_data[len(alias) + 1:]

                if "Position" in stripped_data:  # Sprawdzanie, czy pakiet zawiera słowo "Position"
                    position_text.config(state=tk.NORMAL)
                    position_text.delete(1.0, tk.END)
                    position_text.insert(tk.END, stripped_data + "\n")
                    position_text.config(state=tk.DISABLED)
                elif "Kp" in stripped_data:  # Sprawdzanie, czy pakiet zawiera słowo "Kp"
                    kp_text.config(state=tk.NORMAL)
                    kp_text.delete(1.0, tk.END)
                    kp_text.insert(tk.END, stripped_data + "\n")
                    kp_text.config(state=tk.DISABLED)
                else:
                    text_box.config(state=tk.NORMAL)
                    text_box.delete(1.0, tk.END)
                    text_box.insert(tk.END, stripped_data + "\n")
                    text_box.config(state=tk.DISABLED)
        except:
            break

def send_parameters(robot_udp_ip, Kp, Ki, Kd, Max_speed, Base_speed, Turn_speed, Threshold):
    # Tworzenie gniazda UDP
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    try:
        # Pobieranie wartości z pól Entry
        kp_value = float(Kp.get())
        ki_value = float(Ki.get())
        kd_value = float(Kd.get())
        max_speed_value = float(Max_speed.get())
        base_speed_value = float(Base_speed.get())
        turn_speed_value = float(Turn_speed.get())
        threshold_value = float(Threshold.get())

        # Wysyłanie parametrów Kp
        message = f"Kp: {kp_value:.2f}"
        sock.sendto(message.encode(), (robot_udp_ip, UDP_PORT))
        # Wysyłanie parametrów Ki
        message = f"Ki: {ki_value:.2f}"
        sock.sendto(message.encode(), (robot_udp_ip, UDP_PORT))
        # Wysyłanie parametrów Kd
        message = f"Kd: {kd_value:.2f}"
        sock.sendto(message.encode(), (robot_udp_ip, UDP_PORT))
        # Wysyłanie parametrów Max_speed
        message = f"Ma: {max_speed_value:.2f}"
        sock.sendto(message.encode(), (robot_udp_ip, UDP_PORT))
        # Wysyłanie parametrów Base_speed
        message = f"Ba: {base_speed_value:.2f}"
        sock.sendto(message.encode(), (robot_udp_ip, UDP_PORT))
        # Wysyłanie parametrów Turn_speed
        message = f"Tu: {turn_speed_value:.2f}"
        sock.sendto(message.encode(), (robot_udp_ip, UDP_PORT))
        # Wysyłanie parametrów Threshold
        message = f"Th: {threshold_value:.2f}"
        sock.sendto(message.encode(), (robot_udp_ip, UDP_PORT))
    except Exception as e:
        print("Błąd podczas wysyłania parametrów:", e)
    finally:
        # Zamykanie gniazda
        sock.close()

def start_application(robot_udp_ip, alias=None):
    global text_box, position_text, kp_text

    # Tworzenie okna głównego
    root = tk.Tk()
    if alias:
        root.title(f"Python Line FollowerV0 - {alias}")
    else:
        root.title(f"Python Line FollowerV0 - {robot_udp_ip}")

    # Tworzenie ramki do umieszczenia przycisku
    frame = tk.Frame(root)
    frame.pack(padx=5, pady=5)

    # Tworzenie przycisków i dodanie do ramki
    cal_button = tk.Button(frame, text="Start calibration!", command=lambda: on_button_click("Cal", robot_udp_ip))
    cal_button.grid(row=0, column=0)

    start_button = tk.Button(frame, text="Start", command=lambda: on_button_click("Start", robot_udp_ip))
    start_button.grid(row=1, column=0)

    stop_button = tk.Button(frame, text="Stop", command=lambda: on_button_click("Stop", robot_udp_ip))
    stop_button.grid(row=1, column=1)

    reset_button = tk.Button(frame, text="Reset Robot", command=lambda: on_button_click("Reset", robot_udp_ip))
    reset_button.grid(row=1, column=2)

    Kp_frame = tk.Frame(frame)
    Kp_frame.grid(row=2, column=0)
    tk.Label(Kp_frame, text="Kp:").grid(row=0, column=0)
    Kp = tk.Entry(Kp_frame)
    Kp.grid(row=0, column=1)

    Ki_frame = tk.Frame(frame)
    Ki_frame.grid(row=2, column=1)
    tk.Label(Ki_frame, text="Ki:").grid(row=0, column=0)
    Ki = tk.Entry(Ki_frame)
    Ki.grid(row=0, column=1)

    Kd_frame = tk.Frame(frame)
    Kd_frame.grid(row=2, column=2)
    tk.Label(Kd_frame, text="Kd:").grid(row=0, column=0)
    Kd = tk.Entry(Kd_frame)
    Kd.grid(row=0, column=1)

    Max_speed_frame = tk.Frame(frame)
    Max_speed_frame.grid(row=3, column=0)
    tk.Label(Max_speed_frame, text="Max_speed:").grid(row=0, column=0)
    Max_speed = tk.Entry(Max_speed_frame)
    Max_speed.grid(row=0, column=1)

    Base_speed_frame = tk.Frame(frame)
    Base_speed_frame.grid(row=3, column=1)
    tk.Label(Base_speed_frame, text="Base_speed:").grid(row=0, column=0)
    Base_speed = tk.Entry(Base_speed_frame)
    Base_speed.grid(row=0, column=1)

    Turn_speed_frame = tk.Frame(frame)
    Turn_speed_frame.grid(row=3, column=2)
    tk.Label(Turn_speed_frame, text="Turn_speed:").grid(row=0, column=0)
    Turn_speed = tk.Entry(Turn_speed_frame)
    Turn_speed.grid(row=0, column=1)

    Threshold_frame = tk.Frame(frame)
    Threshold_frame.grid(row=4, column=0)
    tk.Label(Threshold_frame, text="Threshold:").grid(row=0, column=0)
    Threshold = tk.Entry(Threshold_frame)
    Threshold.grid(row=0, column=1)

    send_params_button = tk.Button(frame, text="Send parameters", command=lambda: send_parameters(
        robot_udp_ip, Kp, Ki, Kd, Max_speed, Base_speed, Turn_speed, Threshold))
    send_params_button.grid(row=6, column=0)

    request_params_button = tk.Button(frame, text="Request parameters",
                                      command=lambda: on_button_click("Params", robot_udp_ip))
    request_params_button.grid(row=6, column=1)

    # Tworzenie pola tekstowego do wyświetlania pakietów UDP
    text_box = tk.Text(root, height=5, width=50)
    text_box.pack(padx=5, pady=5)
    text_box.config(state=tk.DISABLED)  # Ustawienie pola tekstowego na nieedytowalne

    # Pole tekstowe dla Position
    position_text = tk.Text(root, height=2, width=50)
    position_text.pack(padx=5, pady=5)
    position_text.config(state=tk.DISABLED)

    # Pole tekstowe dla Kp
    kp_text = tk.Text(root, height=2, width=50)
    kp_text.pack(padx=5, pady=5)
    kp_text.config(state=tk.DISABLED)

    # Uruchomienie funkcji odbierającej pakiety UDP w osobnym wątku
    thread = threading.Thread(target=receive_udp_packets, args=(robot_udp_ip, position_text, kp_text, text_box, alias))
    thread.daemon = True  # Ustawienie wątku jako wątek demoniczny
    thread.start()

    # Rozpoczęcie głównej pętli programu
    root.mainloop()

# Tworzenie okna do wprowadzania adresu IP
def open_ip_window():
    ip_window = tk.Tk()
    ip_window.title("Podaj adres IP robota")

    ip_frame = tk.Frame(ip_window)
    ip_frame.pack(padx=5, pady=5)

    tk.Label(ip_frame, text="Adres IP robota:").grid(row=0, column=0)
    ip_entry = tk.Entry(ip_frame)
    ip_entry.grid(row=0, column=1)

    tk.Label(ip_frame, text="Alias robota (opcjonalnie):").grid(row=1, column=0)
    alias_entry = tk.Entry(ip_frame)
    alias_entry.grid(row=1, column=1)

    def start_app():
        robot_udp_ip = ip_entry.get()
        alias = alias_entry.get()
        if is_valid_ip(robot_udp_ip):
            start_application(robot_udp_ip, alias)
        else:
            ip_entry.config(bg="red")  # Zmiana koloru tła pola wprowadzania na czerwony
            ip_entry.delete(0, tk.END)  # Wyczyszczenie pola wprowadzania
            ip_entry.insert(0, "Nieprawidłowy adres IP")  # Wyświetlenie komunikatu
            ip_entry.after(2000, lambda: ip_entry.config(bg="white"))  # Powrót koloru tła do białego po 2 sekundach

    start_button = tk.Button(ip_window, text="Start", command=start_app)
    start_button.pack(padx=5, pady=5)

    # Uruchomienie głównej pętli programu dla okna do wprowadzania adresu IP
    ip_window.mainloop()

# Create the global socket once
create_global_socket()
open_ip_window()
