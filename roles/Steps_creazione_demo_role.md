Per creare un ruolo Ansible semplice che soddisfi le tue esigenze, segui questi passaggi. Questo esempio è perfetto per una demo, dato che crea una directory e un file con un messaggio specifico.

### 1\. Struttura del Ruolo Ansible

La prima cosa da fare è creare la struttura di directory standard per il tuo ruolo. Puoi usare il comando `ansible-galaxy init` che lo fa automaticamente per te.

```bash
ansible-galaxy init my_demo_role
```

Questo comando creerà la seguente struttura di cartelle e file:

```
my_demo_role/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── README.md
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
└── vars/
    └── main.yml
```

Per questo caso d'uso, lavoreremo principalmente nelle cartelle `tasks/` e `files/`.

-----

### 2\. Creazione del File

Crea il file che vuoi copiare. Visto che il contenuto è semplice, puoi farlo direttamente.

1.  Vai nella cartella `files/` del tuo ruolo:
    `cd my_demo_role/files/`
2.  Crea un file chiamato `demo_file.txt` e aggiungi il contenuto "Hello from ansible role\!".

<!-- end list -->

```bash
echo "Hello from ansible role!" > demo_file.txt
```

-----

### 3\. Definizione dei Task

Ora definiamo i task che il ruolo eseguirà. I task vengono scritti nel file `tasks/main.yml`.

1.  Apri il file `my_demo_role/tasks/main.yml` con un editor di testo.
2.  Aggiungi i seguenti task, che utilizzeranno i moduli **`file`** e **`copy`**.

<!-- end list -->

```yaml
---
- name: Crea la directory di destinazione
  ansible.builtin.file:
    path: /tmp/created_by_ansible_role
    state: directory
    mode: '0755'

- name: Copia il file di demo nella nuova directory
  ansible.builtin.copy:
    src: demo_file.txt
    dest: /tmp/created_by_ansible_role/demo_file.txt
    mode: '0644'
```

  * **Task 1**: Usa il modulo `ansible.builtin.file` per creare la directory `/tmp/created_by_ansible_role`. L'opzione `state: directory` assicura che il percorso sia una cartella, e `mode: '0755'` imposta i permessi standard.
  * **Task 2**: Usa il modulo `ansible.builtin.copy` per copiare il file `demo_file.txt` dalla cartella `files/` del ruolo alla directory di destinazione. Ansible sa automaticamente dove trovare i file nella cartella `files/`, quindi non devi specificare il percorso completo. L'opzione `mode: '0644'` imposta i permessi del file copiato.

-----

### 4\. Creazione del Playbook 

Infine, crea un semplice playbook che esegua il tuo ruolo.

1.  Esci dalla cartella del ruolo e crea un nuovo file chiamato `playbook.yml`.
2.  Aggiungi il seguente contenuto, che fa riferimento al ruolo che hai appena creato.

<!-- end list -->

```yaml
---
- name: Playbook che richiama il ruolo (versione 1)
  hosts: all
  gather_facts: false
  tasks:
    - name: Esecuzione del ruolo
      ansible.builtin.include_role:
        name: my_demo_role
```


✅ Caratteristiche della version 1 con `ansible.builtin.include_role:`:

Richiami il ruolo dentro la sezione tasks.

Puoi includere il ruolo condizionalmente, con when:, oppure passare variabili in quel punto.

È più flessibile: puoi decidere dove nel flusso delle task includere il ruolo.

Esempio:
```yaml
- name: Esegui ruolo solo su Fedora
  ansible.builtin.include_role:
    name: my_demo_role
  when: ansible_distribution == "Fedora"
```
👉 Utile se vuoi controllo granulare.


Oppure anche

```yaml
- name: Playbook che richiama il ruolo (versione 2)
  hosts: all
  become: yes
  roles:
    - my_demo_role
```

✅ Caratteristiche:

Sintassi più semplice e dichiarativa.

Esegui il ruolo sempre, subito dopo il gather_facts e prima di qualsiasi task definita nella stessa play.

Non hai la flessibilità di “metterlo in mezzo” ad altre task.

👉 Utile se vuoi un playbook lineare, tipo “questo play usa questi ruoli”.


🔹 In sintesi

Usa roles: → quando vuoi dichiarare che una play è composta da uno o più ruoli (pulito, leggibile).

Usa include_role → quando ti serve più controllo (condizioni, ordine specifico dentro altre task, passaggio variabili in quel punto).


-----

### 5\. Esecuzione

Per eseguire il playbook, usa il comando `ansible-playbook`.

```bash
ansible-playbook -i <inventory_file>, playbook.yml
```

Alla fine dell'esecuzione, se tutto è andato a buon fine, puoi verificare che la directory e il file siano stati creati correttamente:

```bash
ls -l /tmp/created_by_ansible_role/
cat /tmp/created_by_ansible_role/demo_file.txt
```

Dovresti vedere la cartella e il file `demo_file.txt` con il contenuto "Hello from ansible role\!".


### 6\. Esecuzione per Ansible AWX.
Il Playbook che chiama il ruolo dovrà risiedere nella directory root del repo, dove sono presenti tutti gli altri playbook.
