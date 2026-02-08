## ЗАДАЧА:

<img width="627" height="427" alt="image" src="https://github.com/user-attachments/assets/4a5d0c24-84b6-4350-ab6e-f75e3e3432ce" />

## РЕШЕНИЕ:

### 1) Проверка наличия файла

```powershell
cd $env:USERPROFILE\Downloads
dir firmware.elf
```

### 2) Извлечение данных из ELF (секции `.eeprom` и `.data`)

Данные были извлечены напрямую из ELF через `pyelftools`:

```powershell
py -c "from elftools.elf.elffile import ELFFile; elf=ELFFile(open('firmware.elf','rb')); 
for name in ('.eeprom','.data'):
 s=elf.get_section_by_name(name);
 print(name+(': NOT FOUND' if not s else f' ({len(s.data())} bytes):'));
 if s: print(' '.join(f'{b:02x}' for b in s.data()));
 print()"
```
<img width="975" height="247" alt="image" src="https://github.com/user-attachments/assets/c7a7cf8b-c780-488d-abc5-7feac20ca091" />

### 3) Определение способа шифрования

В секции `.eeprom` обнаружены байты зашифрованной строки, а в `.data` — ключ из 5 байт. По структуре данных сделан вывод, что используется XOR-шифрование с повторяющимся ключом:

[
plain[i] = cipher[i] \oplus key[i \bmod 5]
]

Последний байт EEPROM равен `00`, что соответствует терминатору строки, поэтому расшифровывались первые 31 байт.

### 4) Расшифровка XOR

```powershell
py -c "e=bytes.fromhex('73 17 56 31 6a 65 18 6e 3f 6a 67 10 25 37 70 44 36 25 08 4a 13 1d 22 12 6a 68 04 79 4e 48 4a 00'); k=bytes.fromhex('37 42 15 7a 2f'); pt=bytes([e[i]^k[i%5] for i in range(31)]); print(pt.decode(errors='replace'))"
```

<img width="1204" height="97" alt="image" src="https://github.com/user-attachments/assets/7ce209ab-5db5-433a-89ec-ff5918b2bd72" />


## Результат

Получен флаг: **DUCKERZ{EEPR0M_st0re$_7hE_Fl4g}**
