# Gorsel-ASCII-Cevirici
Gorsel ASCII Cevirici


Exe olarak indirmek için : https://www.mediafire.com/file/e3db85omm9jlq6n/ASCII_Art_Studio.exe/file

YADA ISTERSEN AŞAĞIDAKİ KODU KENDİN BUILD ALIP EXE ALABİLİRSİN

```
import tkinter as tk
from tkinter import filedialog, ttk, colorchooser
from PIL import Image
import os

class ASCIIArtConverter:
    def __init__(self, root):
        self.root = root
        self.root.title("ASCII Art Studio Pro")
        self.root.geometry("1200x800")
        self.root.configure(bg="#2C3E50")  # Koyu mavi arka plan
        
        # Stil ayarları
        style = ttk.Style()
        style.theme_use('clam')  # Modern tema
        style.configure("Custom.TFrame", background="#34495E")
        style.configure("Custom.TButton", 
                       padding=10,
                       font=('Helvetica', 10, 'bold'),
                       background="#3498DB",
                       foreground="white")
        style.configure("Title.TLabel", 
                       font=('Helvetica', 16, 'bold'),
                       foreground="#ECF0F1",
                       background="#34495E")
        style.configure("TLabelframe", 
                       background="#34495E",
                       foreground="#ECF0F1")
        style.configure("TLabelframe.Label", 
                       foreground="#ECF0F1",
                       background="#34495E",
                       font=('Helvetica', 11, 'bold'))
        
        # ASCII karakter setleri
        self.ASCII_SETS = {
            "Standart": "@%#*+=-:. ",
            "Detaylı": "$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/\\|()1{}[]?-_+~<>i!lI;:,\"^`'. ",
            "Basit": "#@$%*+-:. ",
            "Sanatsal": "♠♣♥♦★☆☎☏⚡☢☣☮☯",
            "Emoji": "😀😎😍😢😡😱🤔💫✨"
        }
        self.ASCII_CHARS = self.ASCII_SETS["Basit"]
        
        # Font boyutu için değişken
        self.current_font_size = 12
        
        # Ana frame
        main_frame = ttk.Frame(root, style="Custom.TFrame")
        main_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)
        
        # Sol panel - Sonuç alanı
        left_panel = ttk.Frame(main_frame, style="Custom.TFrame")
        left_panel.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 10))
        
        # Başlık
        title_frame = ttk.Frame(left_panel, style="Custom.TFrame")
        title_frame.pack(fill=tk.X, pady=(0, 20))
        title_label = ttk.Label(title_frame, 
                              text="ASCII Art Studio Pro",
                              style="Title.TLabel")
        title_label.pack()
        
        # Sonuç alanı
        result_frame = ttk.LabelFrame(left_panel, 
                                    text="ASCII Sanat Galerisi",
                                    padding="15")
        result_frame.pack(fill=tk.BOTH, expand=True)
        
        # Metin alanı ve scrollbar
        text_frame = ttk.Frame(result_frame)
        text_frame.pack(fill=tk.BOTH, expand=True)
        
        self.text_area = tk.Text(text_frame,
                                wrap=tk.NONE,
                                font=("Courier", self.current_font_size),
                                bg="#000000",
                                fg="#FF8080",
                                insertbackground="#ECF0F1",
                                relief="flat",
                                padx=10,
                                pady=10)
        
        scrollbar_y = ttk.Scrollbar(text_frame, orient=tk.VERTICAL)
        scrollbar_x = ttk.Scrollbar(text_frame, orient=tk.HORIZONTAL)
        
        self.text_area.configure(yscrollcommand=scrollbar_y.set,
                               xscrollcommand=scrollbar_x.set)
        scrollbar_y.configure(command=self.text_area.yview)
        scrollbar_x.configure(command=self.text_area.xview)
        
        scrollbar_y.pack(side=tk.RIGHT, fill=tk.Y)
        scrollbar_x.pack(side=tk.BOTTOM, fill=tk.X)
        self.text_area.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        
        # Alt butonlar
        button_frame = ttk.Frame(left_panel, style="Custom.TFrame")
        button_frame.pack(fill=tk.X, pady=(15, 0))
        
        for text, command in [
            ("💾 Kaydet", self.save_result),
            ("🗑️ Temizle", lambda: self.text_area.delete(1.0, tk.END)),
            ("📋 Kopyala", self.copy_to_clipboard)
        ]:
            btn = ttk.Button(button_frame,
                           text=text,
                           command=command,
                           style="Custom.TButton")
            btn.pack(side=tk.LEFT, padx=5, pady=5)

        # Sağ panel - Ayarlar
        right_panel = ttk.Frame(main_frame, style="Custom.TFrame")
        right_panel.pack(side=tk.RIGHT, fill=tk.BOTH, padx=(10, 0))
        
        # Dosya seçimi
        file_frame = ttk.LabelFrame(right_panel,
                                  text="Görsel Seçimi",
                                  padding="10")
        file_frame.pack(fill=tk.X, pady=(0, 10))
        
        self.file_path = tk.StringVar()
        ttk.Entry(file_frame,
                 textvariable=self.file_path).pack(fill=tk.X,
                                                  pady=(5,5))
        ttk.Button(file_frame,
                  text="🖼️ Görsel Seç",
                  command=self.browse_file,
                  style="Custom.TButton").pack(fill=tk.X)
        
        # Ayarlar
        settings_frame = ttk.LabelFrame(right_panel,
                                      text="Sanat Ayarları",
                                      padding="10")
        settings_frame.pack(fill=tk.X, pady=(10,0))
        
        # Genişlik ayarı
        ttk.Label(settings_frame,
                 text="Genişlik:",
                 background="#34495E",
                 foreground="#ECF0F1").pack(fill=tk.X)
        self.width_var = tk.StringVar(value="300")
        ttk.Entry(settings_frame,
                 textvariable=self.width_var).pack(fill=tk.X,
                                                  pady=(5,10))
        
        # Karakter seti seçimi
        ttk.Label(settings_frame,
                 text="ASCII Seti:",
                 background="#34495E",
                 foreground="#ECF0F1").pack(fill=tk.X)
        self.char_set_var = tk.StringVar(value="Basit")
        char_set_combo = ttk.Combobox(settings_frame,
                                    textvariable=self.char_set_var,
                                    values=list(self.ASCII_SETS.keys()))
        char_set_combo.pack(fill=tk.X, pady=(5,10))
        char_set_combo.bind('<<ComboboxSelected>>', self.update_ascii_chars)
        
        # Renk seçimi
        self.text_color = "#FF8080"
        self.bg_color = "#000000"
        ttk.Button(settings_frame,
                  text="🎨 Metin Rengi",
                  command=self.choose_text_color,
                  style="Custom.TButton").pack(fill=tk.X,
                                             pady=5)
        ttk.Button(settings_frame,
                  text="🖌️ Arkaplan Rengi",
                  command=self.choose_bg_color,
                  style="Custom.TButton").pack(fill=tk.X,
                                             pady=5)
        
        # Seçenekler
        self.invert_var = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame,
                       text="Renkleri Ters Çevir",
                       variable=self.invert_var,
                       style="TCheckbutton").pack(fill=tk.X,
                                                pady=5)
        self.bold_var = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame,
                       text="Kalın Yazı",
                       variable=self.bold_var,
                       style="TCheckbutton").pack(fill=tk.X,
                                                pady=5)
        
        # Yazı boyutu kontrolü
        size_frame = ttk.LabelFrame(settings_frame,
                                  text="Yazı Boyutu",
                                  padding="10")
        size_frame.pack(fill=tk.X, pady=10)
        
        self.zoom_scale = ttk.Scale(size_frame,
                                  from_=1,
                                  to=72,
                                  orient="horizontal",
                                  value=self.current_font_size,
                                  command=self.on_zoom_scale)
        self.zoom_scale.pack(fill=tk.X)
        
        # Dönüştür butonu
        ttk.Button(right_panel,
                  text="✨ Dönüştür",
                  command=self.convert,
                  style="Custom.TButton").pack(fill=tk.X,
                                             pady=15)

    def on_zoom_scale(self, value):
        self.current_font_size = int(float(value))
        self.update_font()

    def zoom_in(self):
        if self.current_font_size < 72:
            self.current_font_size += 1
            self.zoom_scale.set(self.current_font_size)
            self.update_font()

    def zoom_out(self):
        if self.current_font_size > 1:
            self.current_font_size -= 1
            self.zoom_scale.set(self.current_font_size)
            self.update_font()

    def update_font(self):
        weight = "bold" if self.bold_var.get() else "normal"
        self.text_area.configure(font=("Courier", self.current_font_size, weight))

    def update_ascii_chars(self, event=None):
        self.ASCII_CHARS = self.ASCII_SETS[self.char_set_var.get()]

    def choose_text_color(self):
        color = colorchooser.askcolor(title="Metin Rengi Seç",
                                    color=self.text_color)
        if color[1]:
            self.text_color = color[1]
            self.text_area.configure(fg=self.text_color)

    def choose_bg_color(self):
        color = colorchooser.askcolor(title="Arkaplan Rengi Seç",
                                    color=self.bg_color)
        if color[1]:
            self.bg_color = color[1]
            self.text_area.configure(bg=self.bg_color)

    def copy_to_clipboard(self):
        self.root.clipboard_clear()
        self.root.clipboard_append(self.text_area.get(1.0, tk.END))

    def browse_file(self):
        file_path = filedialog.askopenfilename(
            filetypes=[
                ("Görsel Dosyaları", "*.png *.jpg *.jpeg *.gif *.bmp"),
                ("Tüm Dosyalar", "*.*")
            ]
        )
        if file_path:
            self.file_path.set(file_path)

    def resize_image(self, image, new_width):
        width, height = image.size
        aspect_ratio = height/width
        new_height = int(aspect_ratio * new_width * 0.55)
        return image.resize((new_width, new_height))

    def grayify(self, image):
        return image.convert("L")

    def pixels_to_ascii(self, image):
        pixels = list(image.getdata())
        ascii_str = ""
        if self.invert_var.get():
            pixels = [255 - p for p in pixels]
        for pixel_value in pixels:
            index = min(pixel_value * (len(self.ASCII_CHARS)-1) // 255,
                       len(self.ASCII_CHARS) - 1)
            ascii_str += self.ASCII_CHARS[index]
        return ascii_str

    def convert(self):
        try:
            self.text_area.delete(1.0, tk.END)
            
            if not self.file_path.get():
                self.text_area.insert(tk.END, "🖼️ Lütfen bir görsel seçin!")
                return
            
            try:
                width = int(self.width_var.get())
                if width <= 0:
                    raise ValueError
            except ValueError:
                self.text_area.insert(tk.END, "⚠️ Geçerli bir genişlik değeri girin!")
                return
            
            image = Image.open(self.file_path.get())
            image = self.resize_image(image, width)
            grayscale_image = self.grayify(image)
            ascii_str = self.pixels_to_ascii(grayscale_image)
            
            img_width = grayscale_image.width
            ascii_img = ""
            for i in range(0, len(ascii_str), img_width):
                ascii_img += ascii_str[i:i+img_width] + "\n"
            
            self.text_area.configure(fg=self.text_color, bg=self.bg_color)
            self.update_font()
            
            self.text_area.insert(tk.END, ascii_img)
            
        except Exception as e:
            self.text_area.insert(tk.END, f"❌ Hata oluştu: {str(e)}")

    def save_result(self):
        if not self.text_area.get(1.0, tk.END).strip():
            return
            
        file_path = filedialog.asksaveasfilename(
            defaultextension=".txt",
            filetypes=[("Metin Dosyası", "*.txt"), ("Tüm Dosyalar", "*.*")]
        )
        if file_path:
            with open(file_path, "w", encoding="utf-8") as f:
                f.write(self.text_area.get(1.0, tk.END))

if __name__ == "__main__":
    root = tk.Tk()
    app = ASCIIArtConverter(root)
    root.mainloop()
```

```
*+*%+%%*+**+*+-*****+**%**%$$$*++-***%*+++%$%*%*++--++++++-++-++++*+*-+++++-++++*+-+++*+*+-+**+*+*+**++++-+++*++-****++**+**+*+*********%%%%*%+%%%*%%%
++$++%%%*%++**++%%**+**++**$%+%$*+-+%%@%++-$$%*+**+++++-+**++**+-+*****+*++*-++++**+*+*+++*++*+++++++++*++++*+*++++++**+*+***++**%+*******%*+%*%*%%***
+%*+***%*%%**+**+%%%%+*%**%%**%$*%+-%$#$*%*+*++*+*%+*%*++***%*%**%%$%%%*%%%***%**%%%%%*****+*+***+++*++++*++*+++*++++*++*****+*%******$%*%*+%****%**%*
+*+**+*+*%*****-+%%$$$$%**%$$@$***+*$$@@@$$*++-++**++***%%%%%%%*+-*++--::::::::::.:-++*%%%%*%*++*++**+*+++++++%*+++*++**+**+*%******%*+%%+**%%**%%%%%*
%***%*+*+******-*%%%+*$%**$$%%%%%%@$$@@@@#%++**++*%%*%%%+-:::............................::-**%%****++++*+*++++*+**+*+******+**********%**%%*%*+*+*%$%
***+**%*+**+%+++**%%+$$*%*$$%$@$%$$@@@@@@@$-++**%*%$*+:........................................:-+*%%*******++*+***+*+++++*******%*****+*****%*%%***%*
**%**+***+**%+*****%*%$%*$$$$$*%*$@@@@@@#@$*++*%%+-:...............................................:+*%%%*******++****+*+++++*+*%****+%%%**%*%%****%*%
@*+**%+%**++*%*+*****%%***%%$$*%@%$$@$$@@@@@%**::.....................................................:-*$*+*+**++++*+*++*+++*******+*****%%+*+*%%**%*
%***+%*%%***++-***%**%*%*%%%$$$$@@$$$$$@@$$$%-...........................................................-*%***++*+*++****+********+*****+*%****%%***%
*%**+%%***+**+++*%**%*%%%%%%$*$@$@$@*%%$@@%:...............................................................-%$%*+++++****++*+++++**++***+%+*******%***
*%*%+*%**+*+*+**+*+%%+%%%*%%$*$$@$$@$%$@%-...................................................................-%$***+*+*++++++*+*+***+***+***%****%****
+*+**+*-%+%+%++++*%*%+%*%%%%%%$%$$$$%$@*:.....................................................................:*$%****+**+++**+*+++**+*+***+****%*****
*$*++*++**++**+******+**+*%**$%%%$%$%$*.........................................................................-%$**%+*+++**+**+*****+***+*%+***%****
*%*+**+***+**+++****+*++%*%***$%%%$@%-...........................................................................:$%****++*+++**++**+*+***++****%%%**+
++*+**+***+*+*++*+++++**%+*%%*$%$@@%:..............................................................................+%****++++++**+**++*****+****%***%*
**+%+++***+**++*++*+++*+++*%%*%$@$+.................................................................................-%%*+++*++++*++%++++**+++*********
++%*+++*+++**++++++++++*+*%**%%$$-...................................................................................:****++++++*++++-++*+*+***+****%+
++*+*-+**+++++++*+++*++*+*+***%$-.................................... ..............................:::...............:******+++++++++*+++*++*++**+***
*+**+++*++++++++++-+*++****+****:....................................::.:....................:..:++::+*:...............:$*++++++++*+++*+++*++%+*+*****
*++*++++****+++++*+**+**%****%$-...................:.:+::-:-*--+**+-:+---.................::--::-++-:+*++...............%%*+++++++++++++*+++*++*+**++*
+***-%+**++++%-+*+-*-++*%***%$%:..................-+++*-++-*+-%%$%%*+*-:::.-:.:::....:-::++-+--+++++-*+++-..............%%*+*++*+***++++**++*++*++**++
++++++++*+++%++*+++*+++***+%%$-...................+**-++*----**%**++--:-:----:-+**+:-*+-+**%++*+**+-++-++-:.............*%+*++*++++++**+**++*+-+++*+%+
++-++*++*+++*+++*+*%++++%****$-..................-*%**+-+%*-+*+%*+--::---+**++%$-%%*%%+++%%$%*%%***+-++-:--:............%%**++++*+++++++*++**+-+*%+++*
*++*++****++--++++++**+*%*%%%@:..............:...%%%%%%*+%**++**%+::--+-++**--:%*+%$*--$$**%***%****++*+-:-:............%%**++++++++*++++++*++++*+****
-+++++*--++++-*++++**+++%%%%%@:................::**++*++%%*++*+--+++-+-+**$*:-.-*-%**+-+%*+:-+++%*++%%%+-:-:............%%**+++*++*-++*+++++++*+*++**+
+++-+*-+*+++++%**-+***+%+**%%$:..............::--++-++-+++-:-*-+-+**+*+***+%+-*+*-****%+-***+**+*+*++*++--::...........:%%+**++++++--+++-+++++*+*++*++
-**+-+-+++*+*++++-++++++**+%*$-..............:--++:-**-+%**+---+::----%-*%:::::-*::::::::+%****+++**-+-+-::............:$*+*++**+++++++*++++++++**+++*
-+++-**--+*++++++**+*+++****%$-............:---:--++*--+%%+++++*++%%*%$%+-*%+*+*%*%$%%%$%%***%%%%%%%%*--:::-::.........:$**++++*+++*+++++++++++++*++++
-**-+++-++--+++++*++-**++%*+*$+...........:-+-:::-++*$$%%%%*+*$@%*$$$*++-*$*-*++$%$$$$@@$%+*%*%@%:-+---**+-:::-::.....:*%*+++**+++-+++++-+-+++-+**+++*
+++++*-+-+++-++*-*+++**+*%***%*...........:::::-+$$%%%%+**+*+*$%**$#$%%+++%+---+++%%%%$$$$**+:-+:.......:-*-::++--....-$***++++*++**+*+++--+++++*++*++
-++--++*-++*%%--+*++++***++**%$-.........:::::-+**-::...:::+*++-+-%@%-+$*-++-++*+-*%+-::::...................----*:...+$+**++++**+*++++-++++-++++-+++*
+*+-+++-+-++**-++++*++***+%****$:......-:..::.:.................:..:-+*-::*--*$$*+--:...............:*+:.....:--+%:...%$%%*++++*++*-+++-+++-*--++*++++
+-++*----+*%+-*++++-+*%++***%+*%%.....:-:::..........:::::...........::...:.:-%$*+:..::............::*$%-:....---*:..-@$%+*%*+++-+*++*++-+++-++++*++++
--++--+----++++--++++*++**++%%%%#*....::-::.......-+*%$$$%*++::..::...::::.:+*%%%*-::-::...........:::--.:...:++-+-..:+:...:***+++-++++--+++-++++++++*
++--++*++-++++-*+++++++*++***-::*%:...:--::.....:+*+::::.........-%$*.::++--+%*%%%+-..:-:....:..:..:::::.::-:+::--:...++:...-%*+++++*+++++--++-*+--+++
++-+-+++*++-+--*+*+**+++++%%...:::....:-:.:-....::....*-.....+$+:..-+...++-+-*%$*---..:-...::...:..::::.:::--+:::::..:*-+:...%%+-*++*++++++-+--+-*-+-+
*-+++++-+*--*+++*++++%++**$-...-**.....:..:---........%%-:.:+@#$-:..--..:..:-*$%+:...:::..:::.....::---::-----::--::.:-+@+...+%+**+-++*++--++++*-+-+++
-+-+++*++++-*%-+++++++*-*+%:...::-:......:+-*+-:.......:---+--:.....--......+*%**-:...:::----:..:::++--::---+::-:-::.:*%$+...:%+++++-+*++-+-+++-++++-+
-+-+++-+-+-++++++++++++****:...+-:-...:.:-++*+*++::................:+:::...:-*%**+-::-::::--::.::::-*%+--+***+--:::...++%+...+%+++**++++++*+-+-+++-+--
-*+++-+*++---+-++++*+*++++%-...***+.....:--+-+*++--:::............:**::...::-*%*%*-:-%+-:+**:.::+*--**+::--++-:-:+:...::-+:..%%++++++-+++++*++++-++-+-
-------+-+-+-+-+-+*+++++++%%...*+:.......::*+**++----++--+--+**--++*-::..::-:-+*++:..-++++$%-+*+****%+-::--::::--:.......+..-%+**++++++--++++-+*--++++
+-++++++-*--++**--+-+*+++***-.:+:.......:::--++*++**++***%++%**++%*+:......:::**--:..:+-++%*+%%*-+*+*+----:-::.::........-.:%%++*+++**-++++-+++-+-++++
----*++----++-**++++++++*-*%%..-..........----+----+***+***%****++-:......:::-**+-:...::++$$----:-+:++-:..:*+.....:.....::.*%*+**+*+++++-++--+---++++-
-*--+-+-++++--*+-+-+++-++-+*%-.:..........::.-+-::--+*++++**%*%++++.......:::+%*++:....:::*@*-%%******+-:::----::.....:-:..*++***+++*-+++*+--++--+++-+
+*+--+*-+++++*+*++*+*++-++++**..::-:......:..--:---%+-+*++****%%**-..:...:---*%*++::......+@$%%$***--%+*+--:--++:.....*:..:%+-++++++++++-+++---++---++
--++-++---++++++--+++++-**-**$-...+:........::-:-+***+++**-%%+%%$+..-+...--%$@*+%**+:..-:..-*%%$@%*++---::-:::-::.....-::.*%**+--++++-+-++++++--+-+-++
-------+*---+-++--**+-++*++++*%:..::.......:::--:--++-+-++:++**%*:.-%-...+*$$$$$$%*+::-**:..:+%%%%%%++-+-::::-::.:....-:.*%**++*--+++-+++---+++--+-+++
-+---+--++-+++++-+++++-*+++++**%:..-........::--:-+--++:---%****-..:-....:++*$%*%+:....::..::::++*+::--::::+::...:....:-*%%**+++*+-*+--+--+++++--+--+-
+++++++-++--+++++++-*+-+-+++*++*%:..........:::::--:-::-++*%++*--:........:++-:::........:++-:-+:-+:::.:::---:-:.....-%%*+***+++*+++++-+-*---+++------
+-++--+-----+++++*+-+++++++*+*++%*+++:......:-:::--:--+*-+*+--++%%:.........:::.......:-*-+++-+%-.::...:.::--++.....:%*+***+*+++++-*++++--------+-+-+-
+-+*-+++++-+++++-++-++-++++*++*++*+%$:.......:::::--::--:+++++*++:........................-%--+++::::..::-+--:......-$*++*+*****-++++-+++++-+*------+-
-+-+++++-*--+--+++--++-+++++++*++***%-.......::.:-::..::-+-:-:-:...........................::+--+-+--:::..---:......*%**+*+**++++-+++++*+++++-+++-++-+
+++-++-+--++-++++-*++++-++++-+++**+*%+........:::::..:--:.......................................:::-+-::..:::::....:%*%*++*+++*+*+++++++++++++-+++*++-
++++-*+++++++++-++++++++-++++++++*++*%:.......:--:-::::...................... ............ .........--:::::::......-%+***+%*+*++*+*+-++++-+--+--++-+-*
--+-***++++-++++*----+--*+++++*++++**%-......:.:-::-+:.....................-%*:.......................--::--:......++--*+****%*+**+++++-++++-++-+++-**
+++++--+*+-+++++-+++*-++++++++++++++*%*........:-::-:....................:.:+%:.......................:+:::--:....-$*--%++**+*++*+--++++++-+-+++-+++++
%++++--+++++++++-+++*++++*+++++++++***$:........:::....................................................-+---:....:%$%+*%%-+*+**+**++--+++++++-++++++++
+++-++*++-++++++-++-*++++++*+++++*+***$+........::.....................:::-:+*++*%::...................:-++:....:$$%***%%**+***++**++++++--++-+--++++*
++**++++++++++-++*+-+++++++++++++**+***$:.......::........::--.........--*@%@@$@$@*-:..::::--:..........:-*:....+@$$%**%%%%++***+**++++*+-+-++++++++++
*****+++++++*++++++-+++++++*++++**+****%*.......::.:.....+++*--:::.:::-::-***%$$%@$+*%$***+--+:......:.:::-....+$%$$%*-%%%%****++%*-++**++-++++-++++++
*+*+**+++++++++*+++*+++++++++*++***+*%+%%..........:....:---+:-+--:...........::::::-*$$$%*+++:::...:--.:.....-$%*%***+***%***+++*++++*+*++*++++-+++++
+**+**+++*++++++++++++++++++++*+++++***%%...................::---+::.................:-%%+::::..::...........:$%%**+*-++++++*+*+++*+++++-+*+--++++++++
**%**%**+%+++-*+*++++++++*++%++**++***+%$.......................:++%*%+..........-:---++-:.::....::.-+-......*$**+-+*+*++-++*++++%*++++++++-+++++++++-
%****+******+++***++++++*+++***+******+%$:..................::..:-+*%$*+%*.:*::*%$%@@%$+--::--:..............%%++%%***%%+*+****+-++++++-++++++++++*+*+
**%**++%%*+**+********++**+++**%****%**$$:..................:-..::*:+%+%#@+%++-$@%%%%*%+:::-+-::.............:+%%$%%**%%+*****+%%++-++++++-++++-++++++
%%%%%*******%*******+******%%%%$%%%%%%$$*........................:-:--+++*@****@$+%*+%+.:::-::.................:-+%$$%%***++++**++++++**++++++*+**+***
%%%%%*%****%************%$%*+********+--.........................:.:+--++**-::-%+:+:-%+::::-:.............:.:......-+%%$$%**++*+++++++++*+++*+*+*++***
$%$%%%%$%%%%%*%***%****%$*.........................................:---+*-:::.-*:%-++:--.:.::.............:.:...::....:-*%%%%%*++*++++++***+**-***+**+
$$$%$%%$%%%%$%%%%*$*%$$@$-......:::--.....:............................:::.-:.:.:+:--....................:::....-+::.......:-+%%%%%**%***%+*****%**+*+
$$$@$%%$$$%%%%$$%%$$%%*-:.....:--::-:.....::...:::..............................:.:...................:::::......::::::........:-+---+++*%******++*+**
$@@@$$@$%%$$@@#@$*+-.........:--::::......:::::::::::.................................................:-:::..........::.:.................-+%%***++*%*
@@#@@@@@@@@@$*+::.............:+-::::.....:-----:::---:................................ .......:::.::.:-:.....:..:...:::.:........--........:**%%%****
#@####@@%*-:..................:---::-:....::-++--------:.:...................................:--:------:::..:.:-:.....::::::.:::::-----:.......:-*%%%*
@@$%*+-:.......................:-:----::...::--+++-:---:::..............................:::.:::--------:::.::::-::...:-----::-+------+-:..........:-*%
-::.............................-+-:-::-:...:::-*+--++--+-:::............:::....:::-:..:..:::---+-----:::::.::::::::::--::--:-++-+-+*-................
.................................*%::--:-::....-+*+-++--++++---:.:....:..::----+--+-.::-++%%**+*+:---::+---::::-::-:-----::---++--+%-.................
.................................:%$:-+:::::...::+++-----++++*+++--::........:::....+%%**%**+-++----:+---:--:--:::-:-+++++----:--**:..................
..................................:%$*-+---+-::..--+++-:-++:--::--+::-:::.......::--***++++-+--:+-----------:::--+-+----*+---*+%%+....................
...................................:+@%+++-+++++::::-++------+-:::-++----::.:-+-*****+-++++-----:++------:+---++-++----+--++*$%*:.....................
.....................................-$$*++-:::++::::--+:-+--------+++++******++++-++**+++-------*+:-++:-----++*+----++*++++%*:.......................
......................................:$@**-::.--:-.:::--++-++-+++--++++++***+-+-++--++-++*--+--+:--++--:+-::+++-+--+-++++%*:.........................
```

