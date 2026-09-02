Я сгенерировал 2 симулятора утилиты DISKPART (Google Gemini сгенерировал файл .py, а Claude AI сгенерировал HTML-файл) и потренировался в использовании этой утилиты.

## Исходный код:
### Google Gemini

```python
import cmd
import re

class FullDiskPartSimulator(cmd.Cmd):
    prompt = 'DISKPART> '
    intro = "Microsoft DiskPart версии 10.0\nРАСШИРЕННЫЙ СИМУЛЯТОР ДЛЯ ОБУЧЕНИЯ (Не вносит реальных изменений в систему)\nВведите 'help' для списка команд."

    def __init__(self):
        super().__init__()
        # В реальном DISKPART размеры указываются в мегабайтах (МБ)
        self.disks = {
            0: {"status": "В сети", "size": 512000, "free": 0, "partitions": [
                {"id": 1, "type": "Основной", "size": 512000, "fs": "NTFS", "letter": "C"}
            ]},
            1: {"status": "В сети", "size": 32768, "free": 32768, "partitions": []}
        }
        self.selected_disk = None
        self.selected_part = None
        self.part_counter = 2

    def do_list(self, arg):
        """Вывод списка объектов: list disk, list partition, list volume"""
        args = arg.lower().split()
        if not args:
            print("Укажите параметр. Например: LIST DISK, LIST PARTITION или LIST VOLUME")
            return
            
        if args[0] == "disk":
            print("\n  Диск ###  Состояние      Размер   Свободно")
            print("  --------  -------------  -------  -------")
            for d_id, d in self.disks.items():
                marker = "*" if self.selected_disk == d_id else " "
                print(f"{marker} Диск {d_id}    {d['status'].ljust(13)} {str(d['size']//1024).rjust(3)} Гбайт  {str(d['free']//1024).rjust(3)} Гбайт")
            print()
            
        elif args[0] in ["partition", "part"]:
            if self.selected_disk is None:
                return print("Диск не выбран. Сначала выберите диск.")
            d = self.disks[self.selected_disk]
            print("\n  Раздел    Тип              Размер")
            print("  --------  ---------------  -------")
            for p in d['partitions']:
                marker = "*" if self.selected_part == p['id'] else " "
                size_str = f"{p['size']//1024} Гбайт" if p['size'] >= 1024 else f"{p['size']} Мбайт"
                print(f"{marker} Раздел {p['id']}  {p['type'].ljust(15)} {size_str.rjust(8)}")
            print()
            
        elif args[0] in ["volume", "vol"]:
            print("\n  Том ###  Имя  Метка        ФС     Тип         Размер   Состояние")
            print("  -------  ---  -----------  -----  ----------  -------  ---------")
            vol_id = 0
            for d_id, d in self.disks.items():
                for p in d['partitions']:
                    marker = "*" if (self.selected_disk == d_id and self.selected_part == p['id']) else " "
                    fs = p.get('fs', 'RAW')
                    letter = p.get('letter', ' ')
                    size_str = f"{p['size']//1024} ГБ" if p['size'] >= 1024 else f"{p['size']} МБ"
                    print(f"{marker} Том {vol_id}    {letter.ljust(3)}                {fs.ljust(5)}  Раздел      {size_str.rjust(7)}  Исправен")
                    vol_id += 1
            print()

    def do_select(self, arg):
        """Выбор объекта: select disk <N>, select partition <N>"""
        args = arg.lower().split()
        if len(args) < 2: return print("Укажите тип и номер. Например: SELECT DISK 1")
            
        try: target_id = int(args[1])
        except ValueError: return print("Номер должен быть числом.")

        if args[0] == "disk":
            if target_id in self.disks:
                self.selected_disk = target_id
                self.selected_part = None
                print(f"Выбран диск {target_id}.")
            else: print("Указанный диск не найден.")
        elif args[0] in ["partition", "part"]:
            if self.selected_disk is None: return print("Сначала выберите диск.")
            parts = [p['id'] for p in self.disks[self.selected_disk]['partitions']]
            if target_id in parts:
                self.selected_part = target_id
                print(f"Выбран раздел {target_id}.")
            else: print("Указанный раздел не найден.")

    def do_clean(self, arg):
        """Очистка диска: clean"""
        if self.selected_disk is None: return print("Диск не выбран.")
        d = self.disks[self.selected_disk]
        d['partitions'] = []
        d['free'] = d['size']
        self.selected_part = None
        print("DiskPart: Очистка диска выполнена успешно.")

    def do_create(self, arg):
        """Создание раздела: create partition primary [size=N]"""
        if self.selected_disk is None: return print("Сначала выберите диск.")
        arg_lower = arg.lower()
        
        if "partition primary" in arg_lower:
            d = self.disks[self.selected_disk]
            if d['free'] == 0: return print("Недостаточно свободного места на диске.")
            
            # Поиск параметра size=
            size_match = re.search(r'size=(\d+)', arg_lower)
            if size_match:
                req_size = int(size_match.group(1))
                if req_size > d['free']: return print("Указанный размер превышает доступное свободное место.")
                part_size = req_size
            else:
                part_size = d['free']
                
            new_part = {"id": self.part_counter, "type": "Основной", "size": part_size, "fs": "RAW", "letter": ""}
            d['partitions'].append(new_part)
            d['free'] -= part_size
            self.selected_part = self.part_counter
            self.part_counter += 1
            print("DiskPart: Указанный раздел успешно создан.")
        else:
            print("Синтаксис: create partition primary [size=<N>]")

    def do_format(self, arg):
        """Форматирование раздела: format fs=<ntfs|fat32> [quick]"""
        if self.selected_part is None: return print("Раздел (том) не выбран.")
        
        fs_match = re.search(r'fs=(ntfs|fat32|exfat)', arg.lower())
        if not fs_match: return print("Укажите файловую систему, например: format fs=ntfs quick")
        
        # Находим и обновляем выбранный раздел
        for p in self.disks[self.selected_disk]['partitions']:
            if p['id'] == self.selected_part:
                p['fs'] = fs_match.group(1).upper()
                print(f"DiskPart: Программа DiskPart успешно отформатировала том.")
                return

    def do_assign(self, arg):
        """Назначение буквы: assign letter=<A-Z>"""
        if self.selected_part is None: return print("Раздел (том) не выбран.")
        
        letter_match = re.search(r'letter=([a-z])', arg.lower())
        if not letter_match: return print("Укажите букву, например: assign letter=x")
        
        new_letter = letter_match.group(1).upper()
        
        for p in self.disks[self.selected_disk]['partitions']:
            if p['id'] == self.selected_part:
                p['letter'] = new_letter
                print(f"DiskPart: Назначение имени диска или точки подключения выполнено успешно.")
                return

    def do_exit(self, arg):
        """Выход из программы: exit"""
        print("Завершение работы DiskPart...")
        return True

    def emptyline(self):
        pass

if __name__ == '__main__':
    FullDiskPartSimulator().cmdloop()
```

## Claude AI

```html
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>DISKPART Simulator</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Courier+Prime:wght@400;700&display=swap');

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: #0c0c0c;
    color: #c0c0c0;
    font-family: 'Courier Prime', 'Courier New', monospace;
    font-size: 14px;
    height: 100vh;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  #titlebar {
    background: #000080;
    color: white;
    padding: 2px 8px;
    font-size: 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    user-select: none;
  }

  #titlebar .title { font-weight: bold; }
  #titlebar .controls span {
    display: inline-block;
    width: 16px;
    height: 14px;
    text-align: center;
    cursor: pointer;
    border: 1px solid #888;
    margin-left: 2px;
    font-size: 10px;
    line-height: 14px;
    background: #c0c0c0;
    color: #000;
  }

  #output {
    flex: 1;
    overflow-y: auto;
    padding: 4px 8px;
    white-space: pre-wrap;
    word-break: break-word;
  }

  #output::-webkit-scrollbar { width: 14px; }
  #output::-webkit-scrollbar-track { background: #c0c0c0; }
  #output::-webkit-scrollbar-thumb { background: #808080; border: 2px solid #c0c0c0; }

  .line { line-height: 1.4; }
  .prompt { color: #c0c0c0; }
  .error { color: #ff6060; }
  .success { color: #c0c0c0; }
  .header { color: #ffffff; font-weight: bold; }
  .highlight { color: #ffff00; }

  #input-line {
    display: flex;
    align-items: center;
    padding: 2px 8px;
    background: #0c0c0c;
    border-top: 1px solid #333;
  }

  #prompt-text {
    color: #c0c0c0;
    white-space: nowrap;
    margin-right: 4px;
  }

  #cmd-input {
    flex: 1;
    background: transparent;
    border: none;
    outline: none;
    color: #c0c0c0;
    font-family: inherit;
    font-size: 14px;
    caret-color: #c0c0c0;
  }

  #status-bar {
    background: #000080;
    color: white;
    padding: 1px 8px;
    font-size: 11px;
    display: flex;
    gap: 16px;
  }
</style>
</head>
<body>

<div id="titlebar">
  <span class="title">Администратор: Командная строка - diskpart</span>
  <div class="controls">
    <span>_</span>
    <span>□</span>
    <span>✕</span>
  </div>
</div>

<div id="output"></div>

<div id="input-line">
  <span id="prompt-text">DISKPART&gt; </span>
  <input id="cmd-input" type="text" autofocus autocomplete="off" spellcheck="false">
</div>

<div id="status-bar">
  <span id="status-disk">Диск не выбран</span>
  <span id="status-part">Раздел не выбран</span>
  <span id="status-vol">Том не выбран</span>
</div>

<script>
// ===================== STATE =====================
const state = {
  selectedDisk: null,
  selectedPartition: null,
  selectedVolume: null,
  disks: [
    {
      id: 0, status: 'В сети', size: '476 ГБ', free: '0 Б', dyn: '', gpt: '',
      partitions: [
        { id: 1, type: 'OEM', size: '100 МБ', offset: '1024 КБ', status: 'Скрытый' },
        { id: 2, type: 'Системный', size: '260 МБ', offset: '101 МБ', status: 'Активен' },
        { id: 3, type: 'Зарезервировано', size: '16 МБ', offset: '361 МБ', status: 'Скрытый' },
        { id: 4, type: 'Основной', size: '474 ГБ', offset: '377 МБ', status: '' },
        { id: 5, type: 'Восстановление', size: '1024 МБ', offset: '474 ГБ', status: 'Скрытый' },
      ]
    },
    {
      id: 1, status: 'В сети', size: '14 ГБ', free: '9 ГБ', dyn: '', gpt: '',
      partitions: [
        { id: 1, type: 'Основной', size: '5 ГБ', offset: '1024 КБ', status: 'Активен' },
      ]
    }
  ],
  volumes: [
    { id: 0, letter: 'C', label: 'Windows', fs: 'NTFS', type: 'Раздел', size: '474 ГБ', status: 'Исправен', info: 'Загрузочный' },
    { id: 1, letter: 'D', label: 'Data', fs: 'NTFS', type: 'Раздел', size: '5 ГБ', status: 'Исправен', info: '' },
    { id: 2, letter: '', label: 'Восстановление', fs: 'NTFS', type: 'Раздел', size: '1024 МБ', status: 'Исправен', info: 'Скрытый' },
    { id: 3, letter: '', label: '', fs: 'FAT32', type: 'Раздел', size: '260 МБ', status: 'Исправен', info: 'Система' },
  ],
  history: [],
  historyIdx: -1,
};

// ===================== OUTPUT =====================
const output = document.getElementById('output');
const input = document.getElementById('cmd-input');

function print(text, cls = '') {
  const div = document.createElement('div');
  div.className = 'line ' + cls;
  div.textContent = text;
  output.appendChild(div);
  output.scrollTop = output.scrollHeight;
}

function printRaw(html) {
  const div = document.createElement('div');
  div.className = 'line';
  div.innerHTML = html;
  output.appendChild(div);
  output.scrollTop = output.scrollHeight;
}

function printTable(headers, rows) {
  // Calculate column widths
  const widths = headers.map((h, i) => Math.max(h.length, ...rows.map(r => (r[i] || '').length)));
  const pad = (s, w) => (s || '').padEnd(w);
  const sep = widths.map(w => '-'.repeat(w)).join('  ');
  print('');
  print('  ' + headers.map((h, i) => pad(h, widths[i])).join('  '), 'header');
  print('  ' + sep);
  for (const row of rows) {
    print('  ' + row.map((c, i) => pad(c, widths[i])).join('  '));
  }
  print('');
}

function updateStatus() {
  const sd = document.getElementById('status-disk');
  const sp = document.getElementById('status-part');
  const sv = document.getElementById('status-vol');
  sd.textContent = state.selectedDisk !== null ? `Диск ${state.selectedDisk} выбран` : 'Диск не выбран';
  sp.textContent = state.selectedPartition !== null ? `Раздел ${state.selectedPartition} выбран` : 'Раздел не выбран';
  sv.textContent = state.selectedVolume !== null ? `Том ${state.selectedVolume} выбран` : 'Том не выбран';
}

// ===================== COMMANDS =====================
const commands = {

  'list disk': () => {
    printTable(
      ['Диск ###', 'Состояние', 'Размер', 'Свободно', 'Дин', 'GPT'],
      state.disks.map(d => [
        (state.selectedDisk === d.id ? '* ' : '  ') + 'Диск ' + d.id,
        d.status, d.size, d.free, d.dyn, d.gpt
      ])
    );
  },

  'list partition': () => {
    if (state.selectedDisk === null) {
      print('Диск не выбран. Используйте команду SELECT DISK.', 'error'); return;
    }
    const disk = state.disks[state.selectedDisk];
    printTable(
      ['Раздел ###', 'Тип', 'Размер', 'Смещение'],
      disk.partitions.map(p => [
        (state.selectedPartition === p.id ? '* ' : '  ') + 'Раздел ' + p.id,
        p.type, p.size, p.offset
      ])
    );
  },

  'list volume': () => {
    printTable(
      ['Том ###', 'Букв', 'Метка', 'Фс', 'Тип', 'Размер', 'Состояние', 'Сведения'],
      state.volumes.map(v => [
        (state.selectedVolume === v.id ? '* ' : '  ') + 'Том ' + v.id,
        v.letter, v.label, v.fs, v.type, v.size, v.status, v.info
      ])
    );
  },

  'detail disk': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    const d = state.disks[state.selectedDisk];
    print('');
    print('ATA Device (Диск ' + d.id + ')');
    print('Тип диска             : SCSI');
    print('Состояние             : ' + d.status);
    print('Путь                  : 0');
    print('Конечное устройство   : ' + d.id);
    print('LUN ID                : 0');
    print('Только чтение         : Нет');
    print('Загрузочный диск      : ' + (d.id === 0 ? 'Да' : 'Нет'));
    print('Номер тома страничного файла: 0');
    print('Диск гибернации       : Нет');
    print('Диск аварийного дампа : Нет');
    print('Кластерный диск       : Нет');
    commands['list volume']();
  },

  'detail partition': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    if (state.selectedPartition === null) { print('Раздел не выбран.', 'error'); return; }
    const disk = state.disks[state.selectedDisk];
    const p = disk.partitions.find(x => x.id === state.selectedPartition);
    if (!p) { print('Раздел не найден.', 'error'); return; }
    print('');
    print('Раздел ' + p.id);
    print('Тип        : ' + p.type);
    print('Скрытый    : ' + (p.status === 'Скрытый' ? 'Да' : 'Нет'));
    print('Активный   : ' + (p.status === 'Активен' ? 'Да' : 'Нет'));
    print('Смещение в байтах: ' + p.offset);
    print('');
    const vol = state.volumes.find(v => v.size === p.size);
    if (vol) { print('Том, находящийся на этом разделе: Том ' + vol.id); }
  },

  'detail volume': () => {
    if (state.selectedVolume === null) { print('Том не выбран.', 'error'); return; }
    const v = state.volumes[state.selectedVolume];
    print('');
    print('Диск ###  Состояние   Размер   Свободно  Дин  GPT');
    print('--------  ---------  -------  --------  ---  ---');
    print('Диск 0     В сети     476 ГБ     0 Б');
    print('');
    print('Только чтение          : Нет');
    print('Скрытый                : ' + (v.info === 'Скрытый' ? 'Да' : 'Нет'));
    print('Нет буквы диска        : ' + (v.letter === '' ? 'Да' : 'Нет'));
    print('Теневая копия          : Нет');
    print('Точка подключения      : Нет');
    print('');
  },

  'select disk': (args) => {
    const n = parseInt(args[0]);
    if (isNaN(n) || n < 0 || n >= state.disks.length) {
      print(`Указанный диск недействителен.`, 'error'); return;
    }
    state.selectedDisk = n;
    state.selectedPartition = null;
    state.selectedVolume = null;
    print(`Теперь выбран диск ${n}.`);
    updateStatus();
  },

  'select partition': (args) => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    const n = parseInt(args[0]);
    const disk = state.disks[state.selectedDisk];
    const p = disk.partitions.find(x => x.id === n);
    if (!p) { print(`Указанный раздел недействителен.`, 'error'); return; }
    state.selectedPartition = n;
    print(`Теперь выбран раздел ${n}.`);
    updateStatus();
  },

  'select volume': (args) => {
    const n = parseInt(args[0]);
    if (isNaN(n) || n < 0 || n >= state.volumes.length) {
      print(`Указанный том недействителен.`, 'error'); return;
    }
    state.selectedVolume = n;
    print(`Теперь выбран том ${n}.`);
    updateStatus();
  },

  'create partition primary': (args) => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    const sizeArg = args.find(a => a.startsWith('size='));
    const size = sizeArg ? sizeArg.split('=')[1] + ' МБ' : '100 МБ';
    const disk = state.disks[state.selectedDisk];
    const newId = disk.partitions.length + 1;
    disk.partitions.push({ id: newId, type: 'Основной', size, offset: '1 МБ', status: '' });
    print(`DiskPart успешно создал указанный раздел.`);
  },

  'create partition extended': (args) => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    const disk = state.disks[state.selectedDisk];
    const newId = disk.partitions.length + 1;
    disk.partitions.push({ id: newId, type: 'Расширенный', size: '50 ГБ', offset: '1 МБ', status: '' });
    print(`DiskPart успешно создал указанный раздел.`);
  },

  'create partition logical': (args) => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    const disk = state.disks[state.selectedDisk];
    const newId = disk.partitions.length + 1;
    disk.partitions.push({ id: newId, type: 'Логический', size: '25 ГБ', offset: '1 МБ', status: '' });
    print(`DiskPart успешно создал указанный раздел.`);
  },

  'delete partition': () => {
    if (state.selectedDisk === null || state.selectedPartition === null) {
      print('Раздел не выбран.', 'error'); return;
    }
    const disk = state.disks[state.selectedDisk];
    disk.partitions = disk.partitions.filter(p => p.id !== state.selectedPartition);
    print(`DiskPart успешно удалил выбранный раздел.`);
    state.selectedPartition = null;
    updateStatus();
  },

  'delete volume': () => {
    if (state.selectedVolume === null) { print('Том не выбран.', 'error'); return; }
    state.volumes = state.volumes.filter(v => v.id !== state.selectedVolume);
    // Re-index
    state.volumes.forEach((v, i) => v.id = i);
    print(`DiskPart успешно удалил том.`);
    state.selectedVolume = null;
    updateStatus();
  },

  'format': (args) => {
    if (state.selectedVolume === null && state.selectedPartition === null) {
      print('Том или раздел не выбран.', 'error'); return;
    }
    const fsArg = args.find(a => a.startsWith('fs='));
    const fs = fsArg ? fsArg.split('=')[1].toUpperCase() : 'NTFS';
    const quick = args.includes('quick');
    if (!['NTFS','FAT32','FAT','exFAT','ReFS'].includes(fs)) {
      print('Указана недопустимая файловая система.', 'error'); return;
    }
    if (!quick) {
      print('Выполняется форматирование...');
      print('  0 процентов завершено.');
      print('  100 процентов завершено.');
    }
    if (state.selectedVolume !== null) {
      state.volumes[state.selectedVolume].fs = fs;
    }
    print(`DiskPart успешно отформатировал том.`);
  },

  'assign': (args) => {
    if (state.selectedVolume === null) { print('Том не выбран.', 'error'); return; }
    const letterArg = args.find(a => a.startsWith('letter='));
    if (!letterArg) { print('Не указана буква диска.', 'error'); return; }
    const letter = letterArg.split('=')[1].toUpperCase();
    state.volumes[state.selectedVolume].letter = letter;
    print(`DiskPart успешно назначил букву диска или точку подключения.`);
  },

  'remove': (args) => {
    if (state.selectedVolume === null) { print('Том не выбран.', 'error'); return; }
    const letterArg = args.find(a => a.startsWith('letter='));
    const letter = letterArg ? letterArg.split('=')[1].toUpperCase() : state.volumes[state.selectedVolume].letter;
    if (state.volumes[state.selectedVolume].letter !== letter) {
      print('Указанная буква диска не назначена данному тому.', 'error'); return;
    }
    state.volumes[state.selectedVolume].letter = '';
    print(`DiskPart успешно удалил букву диска или точку подключения.`);
  },

  'active': () => {
    if (state.selectedDisk === null || state.selectedPartition === null) {
      print('Раздел не выбран.', 'error'); return;
    }
    const disk = state.disks[state.selectedDisk];
    const p = disk.partitions.find(x => x.id === state.selectedPartition);
    if (!p) { print('Раздел не найден.', 'error'); return; }
    disk.partitions.forEach(x => { if (x.status === 'Активен') x.status = ''; });
    p.status = 'Активен';
    print(`DiskPart помечал текущий раздел как активный.`);
  },

  'inactive': () => {
    if (state.selectedDisk === null || state.selectedPartition === null) {
      print('Раздел не выбран.', 'error'); return;
    }
    const disk = state.disks[state.selectedDisk];
    const p = disk.partitions.find(x => x.id === state.selectedPartition);
    if (p) p.status = '';
    print(`DiskPart успешно пометил текущий раздел как неактивный.`);
  },

  'clean': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    state.disks[state.selectedDisk].partitions = [];
    print(`DiskPart успешно очистил диск.`);
    state.selectedPartition = null;
    updateStatus();
  },

  'extend': (args) => {
    if (state.selectedVolume === null && state.selectedPartition === null) {
      print('Том или раздел не выбран.', 'error'); return;
    }
    print(`DiskPart успешно расширил том.`);
  },

  'shrink': (args) => {
    if (state.selectedVolume === null && state.selectedPartition === null) {
      print('Том или раздел не выбран.', 'error'); return;
    }
    const desiredArg = args.find(a => a.startsWith('desired='));
    const desired = desiredArg ? desiredArg.split('=')[1] : '1024';
    print(`DiskPart успешно сжал том на: ${desired} МБ`);
  },

  'convert gpt': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    if (state.disks[state.selectedDisk].gpt === 'GPT') {
      print('Указанный диск уже является GPT.', 'error'); return;
    }
    state.disks[state.selectedDisk].gpt = 'GPT';
    state.disks[state.selectedDisk].partitions = [];
    print(`DiskPart успешно преобразовал выбранный диск в формат GPT.`);
    state.selectedPartition = null;
    updateStatus();
  },

  'convert mbr': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    state.disks[state.selectedDisk].gpt = '';
    state.disks[state.selectedDisk].partitions = [];
    print(`DiskPart успешно преобразовал выбранный диск в формат MBR.`);
    state.selectedPartition = null;
    updateStatus();
  },

  'convert dynamic': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    state.disks[state.selectedDisk].dyn = 'Дин';
    print(`DiskPart успешно преобразовал выбранный диск в динамический.`);
  },

  'convert basic': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    state.disks[state.selectedDisk].dyn = '';
    print(`DiskPart успешно преобразовал выбранный диск в базовый.`);
  },

  'rescan': () => {
    print('');
    print('Выполняется повторная проверка...');
    print('');
    print('DiskPart успешно выполнил повторную проверку конфигурации дисков.');
  },

  'online disk': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    state.disks[state.selectedDisk].status = 'В сети';
    print(`DiskPart успешно перевёл выбранный диск в режим "В сети".`);
  },

  'offline disk': () => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    state.disks[state.selectedDisk].status = 'Не в сети';
    print(`DiskPart успешно перевёл выбранный диск в режим "Не в сети".`);
  },

  'attributes disk': (args) => {
    if (state.selectedDisk === null) { print('Диск не выбран.', 'error'); return; }
    print('Текущие атрибуты диска:');
    print('Только чтение  : Нет');
    print('Загрузочный    : Да');
    print('Страничный файл: Нет');
    print('Аварийный дамп : Нет');
    print('Кластерный     : Нет');
    print('');
  },

  'attributes volume': () => {
    if (state.selectedVolume === null) { print('Том не выбран.', 'error'); return; }
    print('Текущие атрибуты тома:');
    print('Только чтение          : Нет');
    print('Скрытый                : Нет');
    print('Нет буквы диска        : Нет');
    print('Теневая копия          : Нет');
    print('');
  },

  'help': () => {
    print('');
    print('Команды Microsoft DiskPart:', 'header');
    print('');
    const cmds = [
      ['LIST DISK', 'Вывод списка дисков'],
      ['LIST PARTITION', 'Вывод списка разделов выбранного диска'],
      ['LIST VOLUME', 'Вывод списка томов'],
      ['SELECT DISK <n>', 'Выбор диска'],
      ['SELECT PARTITION <n>', 'Выбор раздела'],
      ['SELECT VOLUME <n>', 'Выбор тома'],
      ['DETAIL DISK', 'Сведения о выбранном диске'],
      ['DETAIL PARTITION', 'Сведения о выбранном разделе'],
      ['DETAIL VOLUME', 'Сведения о выбранном томе'],
      ['CREATE PARTITION PRIMARY [size=<n>]', 'Создание основного раздела'],
      ['CREATE PARTITION EXTENDED', 'Создание расширенного раздела'],
      ['CREATE PARTITION LOGICAL', 'Создание логического раздела'],
      ['DELETE PARTITION', 'Удаление выбранного раздела'],
      ['DELETE VOLUME', 'Удаление выбранного тома'],
      ['FORMAT [fs=NTFS|FAT32|exFAT] [quick]', 'Форматирование тома'],
      ['ASSIGN letter=<X>', 'Назначение буквы диска'],
      ['REMOVE letter=<X>', 'Удаление буквы диска'],
      ['ACTIVE', 'Пометить раздел как активный'],
      ['INACTIVE', 'Пометить раздел как неактивный'],
      ['CLEAN', 'Очистить диск (удалить все разделы)'],
      ['EXTEND', 'Расширить том'],
      ['SHRINK [desired=<n>]', 'Сжать том (МБ)'],
      ['CONVERT GPT', 'Конвертировать диск в GPT'],
      ['CONVERT MBR', 'Конвертировать диск в MBR'],
      ['CONVERT DYNAMIC', 'Конвертировать в динамический диск'],
      ['CONVERT BASIC', 'Конвертировать в базовый диск'],
      ['RESCAN', 'Повторная проверка дисков'],
      ['ONLINE DISK', 'Перевести диск в режим "В сети"'],
      ['OFFLINE DISK', 'Перевести диск в режим "Не в сети"'],
      ['ATTRIBUTES DISK', 'Показать атрибуты диска'],
      ['ATTRIBUTES VOLUME', 'Показать атрибуты тома'],
      ['EXIT', 'Выход из DiskPart'],
    ];
    const w = Math.max(...cmds.map(c => c[0].length));
    for (const [cmd, desc] of cmds) {
      print('  ' + cmd.padEnd(w + 2) + desc);
    }
    print('');
  },

  'exit': () => {
    print('');
    print('DiskPart завершил работу.');
  },
};

// ===================== INPUT HANDLING =====================
function execute(raw) {
  const line = raw.trim();
  if (!line) return;

  // Echo input
  print('');
  print('DISKPART> ' + line, 'prompt');
  print('');

  // History
  state.history.unshift(line);
  state.historyIdx = -1;

  // Parse
  const lower = line.toLowerCase();
  const parts = lower.split(/\s+/);

  if (lower === 'exit') {
    commands['exit']();
    return;
  }

  // Try to match commands with up to 3 word keys
  let matched = false;
  for (let len = 3; len >= 1; len--) {
    const key = parts.slice(0, len).join(' ');
    if (commands[key]) {
      commands[key](parts.slice(len));
      matched = true;
      break;
    }
  }

  if (!matched) {
    print(`"${parts[0].toUpperCase()}" не является допустимой командой DiskPart.`, 'error');
    print('Введите HELP для вывода списка доступных команд.', 'error');
  }
}

input.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    const val = input.value;
    input.value = '';
    execute(val);
  } else if (e.key === 'ArrowUp') {
    e.preventDefault();
    if (state.historyIdx < state.history.length - 1) {
      state.historyIdx++;
      input.value = state.history[state.historyIdx];
    }
  } else if (e.key === 'ArrowDown') {
    e.preventDefault();
    if (state.historyIdx > 0) {
      state.historyIdx--;
      input.value = state.history[state.historyIdx];
    } else {
      state.historyIdx = -1;
      input.value = '';
    }
  }
});

// Click anywhere to focus
document.getElementById('output').addEventListener('click', () => input.focus());

// ===================== STARTUP =====================
print('');
print('Microsoft DiskPart версия 10.0.19041.964', 'header');
print('');
print('Copyright (C) Microsoft Corporation.');
print('На компьютере: DESKTOP-SIM');
print('');
print('Введите HELP для вывода списка доступных команд.');
print('');
</script>
</body>
</html>

```
