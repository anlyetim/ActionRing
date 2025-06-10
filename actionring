#!/usr/bin/env python3

import gi
gi.require_version("Gtk", "4.0")
gi.require_version("Adw", "1")
from gi.repository import Gtk, Gdk, Adw, GLib
import math
import subprocess
import json
import os

BASE_DIR = os.path.dirname(os.path.abspath(__file__))

def load_settings():
    path = os.path.join(BASE_DIR, "settings.json")
    with open(path, "r") as f:
        return json.load(f)

def save_settings(data):
    path = os.path.join(BASE_DIR, "settings.json")
    with open(path, "w") as f:
        json.dump(data, f, indent=4)

settings = load_settings()

def get_icon_path(name):
    return os.path.join(BASE_DIR, "icons", name)

BUTTONS = []
for btn in settings["buttons"]:
    icon_path = get_icon_path(btn["icon"]) if btn["icon"] else None
    BUTTONS.append({
        "name": btn["name"],
        "command": btn["command"],
        "icon": icon_path,
        "enabled": btn.get("enabled", True)  # Burada enabled kullandık
    })


def is_muted():
    try:
        result = subprocess.run(
            ["pactl", "get-sink-mute", "@DEFAULT_SINK@"],
            capture_output=True,
            text=True
        )
        return "yes" in result.stdout.lower()
    except Exception:
        return False

class ActionRing(Adw.Application):
    def __init__(self):
        super().__init__(application_id="dev.moksha.ActionRing")
        self.connect("activate", self.on_activate)
        self.button_widgets = []
        self.settings = settings
        self.timeout_id = None

        # Menü durumları için flagler
        self.menu_state = "main"  # main, settings, select_button, icon_selector
        self.selected_button_index = None

    def on_activate(self, app):
        self.bg_window = Gtk.ApplicationWindow(application=app)
        self.bg_window.set_decorated(False)
        self.bg_window.set_default_size(1920, 1080)
        self.bg_window.fullscreen()
        self.bg_window.set_opacity(0.0)

        self.ui_window = Gtk.ApplicationWindow(application=app)
        self.ui_window.set_decorated(False)
        self.ui_window.set_resizable(False)
        self.ui_window.set_default_size(400, 400)
        self.ui_window.set_title("Action Ring")

        css_provider = Gtk.CssProvider()
        css_provider.load_from_data(b"""
        button {
            background-color: black;
            border-radius: 30px;
            font-size: 20px;
            color: white;
            box-shadow: 0 2px 5px rgba(0,0,0,0);
            outline: none;
        }
        button:hover {
            border: 3px solid white;
        }
        button.muted {
            background-color: #ffe6e6;
        }
        """)
        Gtk.StyleContext.add_provider_for_display(
            Gdk.Display.get_default(),
            css_provider,
            Gtk.STYLE_PROVIDER_PRIORITY_APPLICATION
        )

        self.fixed = Gtk.Fixed()
        self.ui_window.set_child(self.fixed)

        self.draw_buttons()

        # Key event controller
        self.controller = Gtk.EventControllerKey.new()
        self.controller.connect("key-pressed", self.on_key_pressed)
        self.ui_window.add_controller(self.controller)

        self.bg_window.present()
        self.ui_window.present()

        self.start_auto_quit_timer()

    def clear_fixed_children(self):
        children = []
        child = self.fixed.get_first_child()
        while child:
            children.append(child)
            child = child.get_next_sibling()
        for c in children:
            self.fixed.remove(c)

    def draw_buttons(self):
        self.clear_fixed_children()
        self.button_widgets.clear()

        # Sadece aktif butonları alıyoruz
        active_buttons = [btn for btn in BUTTONS if btn.get("enabled", True)]

        radius = 130
        center_x = 200
        center_y = 200
        total = len(active_buttons)
        muted = is_muted()

        for i, btn_data in enumerate(active_buttons):
            angle = (2 * math.pi * i) / total
            x = center_x + radius * math.cos(angle) - 30
            y = center_y + radius * math.sin(angle) - 30

            btn = Gtk.Button()
            btn.set_focusable(False)
            btn.set_size_request(60, 60)

            label = btn_data["name"]
            icon_path = btn_data["icon"]

            if label == "Mute":
                if muted:
                    icon_file = get_icon_path("mute.png")
                    btn.get_style_context().add_class("muted")
                else:
                    icon_file = get_icon_path("volume.png")
            else:
                icon_file = icon_path

            image = Gtk.Image.new_from_file(icon_file)
            btn.set_child(image)

            btn.connect("clicked", lambda b, c=btn_data["command"]: (subprocess.Popen(c), self.do_quit()))
            self.fixed.put(btn, int(x), int(y))
            self.button_widgets.append((btn, btn_data))

    def start_auto_quit_timer(self):
        if self.timeout_id:
            GLib.source_remove(self.timeout_id)
        self.timeout_id = GLib.timeout_add_seconds(5, self.do_quit)

    def on_key_pressed(self, controller, keyval, keycode, state):
        key = Gdk.keyval_name(keyval)
        key_lower = key.lower() if key else ""

        # ESC ile tüm menüleri kapat
        if key == "Escape":
            if self.menu_state != "main":
                self.menu_state = "main"
                self.selected_button_index = None
                self.clear_fixed_children()
                self.draw_buttons()
                self.ui_window.set_title("Action Ring")
                self.start_auto_quit_timer()
                return True
            else:
                self.do_quit()
                return True

        if self.menu_state == "main":
            if key_lower == "s":
                self.open_settings_menu()
                return True
            self.do_quit()
            return True

        elif self.menu_state == "settings":
            # Settings menüde
            if key_lower == "c":  # Change icon seçildi
                self.open_select_button_menu()
                return True
            elif key_lower == "q":
                # Quit ayarı
                self.do_quit()
                return True

        elif self.menu_state == "select_button":
            # Burada sayısal tuşlarla seçim yapabiliriz
            if key.isdigit():
                idx = int(key) - 1
                if 0 <= idx < len(BUTTONS):
                    self.selected_button_index = idx
                    self.open_icon_selector()
                    return True
            elif key_lower == "q":
                # Menüden çık
                self.menu_state = "settings"
                self.open_settings_menu()
                return True

        elif self.menu_state == "icon_selector":
            # İcon seçme ekranı ESC ile kapanır, buraya istersen ek fonksiyon ekleyebilirsin
            if key_lower == "q":
                # İcon seçimi iptal
                self.menu_state = "settings"
                self.selected_button_index = None
                self.open_settings_menu()
                return True

        return True

    def open_settings_menu(self):
        self.menu_state = "settings"
        if self.timeout_id:
            GLib.source_remove(self.timeout_id)
            self.timeout_id = None

        self.clear_fixed_children()

        window_width = 400
        button_width = 200
        center_x = (window_width - button_width) // 2
        start_y = 100
        gap = 80  # Butonlar arası dikey mesafe

        # ESC bilgisi
        info_label = Gtk.Label(label="(ESC) to close settings menu X")
        info_label.set_hexpand(True)
        info_label.set_halign(Gtk.Align.CENTER)
        self.fixed.put(info_label, 100, 40)  # X=100, Y=40 gibi üstte bir yerde

        btn_add_remove = Gtk.Button(label="Manage Buttons")
        btn_add_remove.set_size_request(button_width, 60)
        btn_add_remove.connect("clicked", lambda b: self.open_add_remove_buttons_menu())
        self.fixed.put(btn_add_remove, center_x, start_y)

        btn_change_icon = Gtk.Button(label="Change Icon")
        btn_change_icon.set_size_request(button_width, 60)
        btn_change_icon.connect("clicked", lambda b: self.open_select_button_menu())
        self.fixed.put(btn_change_icon, center_x, start_y + gap)

        btn_quit = Gtk.Button(label="Quit Ring(Q)")
        btn_quit.set_size_request(button_width, 60)
        btn_quit.connect("clicked", lambda b: self.do_quit())
        self.fixed.put(btn_quit, center_x, start_y + 2 * gap)

        self.ui_window.set_title("Settings Menu")


    def open_add_remove_buttons_menu(self):
        self.menu_state = "add_remove_buttons"
        self.clear_fixed_children()

        center_x, center_y = 200, 100

        label = Gtk.Label(label="Add / Remove Buttons")
        label.set_hexpand(True)
        label.set_halign(Gtk.Align.CENTER)
        self.fixed.put(label, center_x - 150, center_y - 80)

        # Checkboxlar
        for i, btn_data in enumerate(BUTTONS):
            check = Gtk.CheckButton(label=btn_data["name"])
            check.set_active(btn_data.get("enabled", True))

            def on_toggled(button, index=i):
                BUTTONS[index]["enabled"] = button.get_active()
                settings["buttons"][index]["enabled"] = button.get_active()
                save_settings(settings)
                #self.draw_buttons()

            check.connect("toggled", on_toggled)
            check.set_size_request(300, 40)
            self.fixed.put(check, center_x - 150, center_y - 20 + i * 50)

        # Back Butonu
        btn_back = Gtk.Button(label="< Back")
        btn_back.set_size_request(300, 60)
        btn_back.connect("clicked", lambda b: self.open_settings_menu())
        self.fixed.put(btn_back, center_x - 150, center_y + 50 + len(BUTTONS) * 50)

        self.ui_window.set_title("Add / Remove Buttons Menu")

    def open_select_button_menu(self):
        self.menu_state = "select_button"
        self.clear_fixed_children()

        center_x, center_y = 200, 200

        # Listeyi sayılarla gösterelim
        for i, btn_data in enumerate(BUTTONS):
            label = f"{i+1}. {btn_data['name']}"
            btn = Gtk.Button(label=label)
            btn.set_size_request(300, 50)
            btn.connect("clicked", lambda b, idx=i: self.select_button_for_icon(idx))
            self.fixed.put(btn, center_x - 150, center_y - 100 + i*60)

        btn_quit = Gtk.Button(label="< Back")
        btn_quit.set_size_request(300, 50)
        btn_quit.connect("clicked", lambda b: self.open_settings_menu())

        quit_y = center_y - 100 + len(BUTTONS) * 60 + 20  # Dinamik quit pozisyonu
        self.fixed.put(btn_quit, center_x - 150, quit_y)

        self.ui_window.set_title("Select Button to Change Icon")


    def select_button_for_icon(self, index):
        self.selected_button_index = index
        self.open_icon_selector()

    def open_icon_selector(self):
        self.menu_state = "icon_selector"
        self.clear_fixed_children()

        center_x, center_y = 200, 200

        icon_names = os.listdir(os.path.join(BASE_DIR, "icons"))
        icon_names = [f for f in icon_names if f.endswith(".png")]

        for i, icon_file in enumerate(icon_names):
            label = icon_file
            btn = Gtk.Button(label=label)
            btn.set_size_request(300, 40)
            btn.connect("clicked", lambda b, icon=icon_file: self.set_icon_for_selected_button(icon))
            self.fixed.put(btn, center_x - 150, center_y - 100 + i*50)

        btn_quit = Gtk.Button(label="< Back")
        btn_quit.set_size_request(300, 50)
        btn_quit.connect("clicked", lambda b: self.open_select_button_menu())

        quit_y = center_y - 100 + len(icon_names) * 50 + 20  # Dinamik pozisyon
        self.fixed.put(btn_quit, center_x - 150, quit_y)

        self.ui_window.set_title("Select Icon")


    def set_icon_for_selected_button(self, icon_file):
        if self.selected_button_index is not None:
            # Update settings and BUTTONS list
            BUTTONS[self.selected_button_index]["icon"] = get_icon_path(icon_file)
            settings["buttons"][self.selected_button_index]["icon"] = icon_file
            save_settings(settings)

            # Geri settings menüye dön
            self.selected_button_index = None
            self.open_settings_menu()

    def do_quit(self):
        if self.timeout_id:
            GLib.source_remove(self.timeout_id)
        self.ui_window.close()
        self.bg_window.close()
        self.quit()

if __name__ == "__main__":
    app = ActionRing()
    app.run()
