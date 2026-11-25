# فایل ۱۷ — Ansible: پیکربندی و مدیریت پیکربندی

**مدت یادگیری**: ۱۰ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۲ (لینوکس/شبکه)، فایل ۱۵ (IaC مفاهیم)

---

## فهرست محتوا

- [فصل ۸۱: مفهوم Configuration Management و معرفی Ansible](#فصل-۸۱-مفهوم-configuration-management-و-معرفی-ansible)
- [فصل ۸۲: ساختار Ansible (Inventory، Playbook، Modules، Tasks)](#فصل-۸۲-ساختار-ansible-inventory-playbook-modules-tasks)
- [فصل ۸۳: Roles و متغیرها (Variables و Vault)](#فصل-۸۳-roles-و-متغیرها-variables-و-vault)
- [فصل ۸۴: Ansible در تیم و ترکیب با Terraform](#فصل-۸۴-ansible-در-تیم-و-ترکیب-با-terraform)
- [فصل ۸۵: بهترین روش‌ها و عیب‌یابی Ansible](#فصل-۸۵-بهترین-روشها-و-عیبیابی-ansible)

---

## فصل ۸۱: مفهوم Configuration Management و معرفی Ansible

### مدمه

ساختن سرور (provisioning) تنها نیمی از کار است — بقیه، **پیکربندی** (Configuration) آن است: نصب نرم‌افزار، تنظیم فایل‌ها، راه‌اندازی سرویس‌ها. **Configuration Management** این را خودکار می‌کند. **Ansible** محبوب‌ترین ابزار این حوزه است و برخلاف بسیاری رقبا، **بدون Agent** کار می‌کند. این فصل چرا و چیست را توضیح می‌دهد.

### مفاهیم اساسی

**Configuration Management (CM) چیست؟**
حفظ وضعیت مطلوب سرورها در طول زمان: «این ۵۰ سرور باید همیشه nginx نصب و فعال داشته باشند». اگر کسی دستی تغییر داد، CM آن را به حالت مطلوب بازمی‌گرداند (idempotency).

**چرا Ansible؟**
- **Agentless:** نیازی به نصب عامل روی هدف نیست — فقط SSH (یا WinRM برای ویندوز).
- **Idempotent:** اجرای چندباره همان نتیجه (ایمن).
- **YAML-based:** خوانا و ساده.
- **Push model:** کنترل‌کننده (controller) تغییرات را به هدف می‌فرستد (در مقابل Pull مدل مثل Puppet).
- **معامل‌پذیر (Agentless + SSH):** سریع راه‌اندازی.

**تفاوت با Terraform:**
- **Terraform:** زیرساخت را **می‌سازد/تخریب می‌کند** (lifecycle کوتاه‌تر منابع).
- **Ansible:** روی سرور **موجود** پیکربندی می‌کند (نصب، تنظیم).
- گاهی هم‌پوشانی دارند (هر دو می‌توانند پکیج نصب کنند) اما هدف متفاوت است.

**جریان کار:**
```
Controller (لپ‌تاپ/CI) → SSH → Target Hosts
                         اجرای playbook (مجموعه taskها)
```

**الگوهای دیگر:** Puppet/Chef (pull-based، agent دارند، پیچیده‌تر)، SaltStack.

### مثال‌های عملی

**۱. نصب Ansible:**

```bash
# لینوکس
sudo apt update && sudo apt install -y ansible
# یا pip
pip install ansible

ansible --version
```

**۲. دستور ساده (ad-hoc):**

```bash
# بررسی ارتباط با همهٔ هاست‌ها
ansible all -i inventory.ini -m ping

# نصب پکیج روی همه
ansible all -i inventory.ini -m apt -a "name=nginx state=present" --become
```

**۳. مفهوم idempotency:**

```bash
# اجرای دوباره همان دستور:
# بار اول: changed (نصب شد)
# بار دوم: ok (قبلاً نصب بود)
```

**۴. مقایسه Terraform vs Ansible (مفهومی):**

```bash
# Terraform: aws_instance ایجاد می‌کند
# Ansible: روی آن instance nginx نصب و تنظیم می‌کند
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | مدل | Agent | کاربرد اصلی |
|---|---|---|---|
| Ansible | Push | نه (SSH) | Config management |
| Terraform | Pull/Apply | نه | Provisioning |
| Puppet | Pull | بله | Config (سازمانی) |
| Chef | Pull | بله | Config (کد-محور) |

**دیدگاه‌ها:**
- Ansible برای تیم‌هایی که می‌خواهند سریع شروع کنند و agent نصب نکنند، ایده‌آل است.
- برای scale بسیار بزرگ (هزاران نود)، Puppet/Chef (pull-based) ممکن است بهتر باشند.
- ترکیب رایج: **Terraform ساخت، Ansible پیکربندی می‌کند**.

**Agentless trade-off:** SSH همه‌جا نیست (مثلاً در برخی containerized environments) — برای آن K8s/GitOps جایگزین است.

### اشتباهات رایج

- **استفاده از Ansible برای provisioning ابر:** Terraform برای آن بهتر است (API بهینه‌تر).
- **فراموش `--become` (sudo):** پکیج نصب نمی‌شود (دسترس).
- **SSH بدون کلید:** ورود با رمز در Ansible دردناک/ناامن است.
- **نوشتن playbook غیرidempotent:** اجرای دوباره خرابی ایجاد می‌کند.
- **فراموش `host_key_checking`:** خطای SSH اولیه.

### نکات مهم و ترفندها

- **SSH keys:** کلید عمومی را روی هدف قرار دهید (بدون رمز).
- **`ansible.cfg`:** تنظیمات محلی (مثل `host_key_checking = False` در آزمایشگاه).
- **`ansible-inventory --list`:** دیدن inventory پارس‌شده.
- **`ansible-vault`:** برای رمزها (فصل ۸۳).
- **Dynamic inventory:** برای ابر (AWS/GCP) که لیست هاست از API می‌آید.
- **محدودسازی اجرا:** `ansible-playbook -l webservers` فقط روی گروه خاص.

### خلاصه فصل

Configuration Management وضعیت مطلوب سرورها (نصب/تنظیم) را حفظ می‌کند. Ansible ابزار agentless (فقط SSH)، idempotent و YAML-based است — برای پیکربندی روی سرورهای موجود عالی. تفاوت کلیدی با Terraform: Terraform می‌سازد، Ansible پیکربندی می‌کند. کلید: SSH keys (بدون رمز)، `--become` برای sudo، نوشتن playbook idempotent، و استفاده از dynamic inventory برای ابر. ترکیب رایج: Terraform + Ansible.

---

## فصل ۸۲: ساختار Ansible (Inventory، Playbook، Modules، Tasks)

### مدمه

برای نوشتن Ansible واقعی، باید اجزای آن را بشناسید: **Inventory** (لیست هاست‌ها)، **Playbook** (طرح اجرا)، **Task** (واحد کار) و **Module** (ابزار هر task). این فصل این اجزا را با مثال عملی می‌سازد.

### مفاهیم اساسی

**Inventory:**
فایلی (INI/YAML) که هاست‌ها و گروه‌هایشان را تعریف می‌کند.

```ini
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com

[all:vars]
ansible_user=ubuntu
```

**Playbook:**
فایل YAML که یک یا چند **Play** دارد. هر Play روی گروهی از هاست‌ها اجرا می‌شود و شامل **Tasks** است.

**Task:**
یک واحد کار که یک **Module** را صدا می‌زند (مثل `apt`، `copy`، `service`).

**Module:**
کدی که کار واقعی را انجام می‌دهد (بیش از ۳۰۰۰ ماژول دارد: apt، yum، copy، template، service، git، docker...).

**Handler:**
Task ویژه که فقط وقتی Task دیگری `notify` کند اجرا می‌شود (مثل restart سرویس پس از تغییر تنظیم).

**Idempotency در ماژول‌ها:** اکثر ماژول‌ها "changed" یا "ok" را گزارش می‌دهند.

**ساختار اجرا:**
```
ansible-playbook -i inventory.ini site.yml
```

### مثال‌های عملی

**۱. Inventory (YAML):**

```yaml
all:
  children:
    webservers:
      hosts:
        web1: { ansible_host: 10.0.0.1 }
        web2: { ansible_host: 10.0.0.2 }
    dbservers:
      hosts:
        db1: { ansible_host: 10.0.0.3 }
  vars:
    ansible_user: ubuntu
```

**۲. Playbook ساده (`site.yml`):**

```yaml
---
- name: Configure webservers
  hosts: webservers
  become: true
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

    - name: Ensure nginx running
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

**۳. اجرا و خروجی:**

```bash
ansible-playbook -i inventory.yml site.yml
# PLAY [Configure webservers] ********************************
# TASK [Install nginx] ************ => changed
# TASK [Copy config] ************** => changed
# RUNNING HANDLER [Restart nginx] * => changed
# PLAY RECAP: web1=changed=3 ...
```

**۴. Task با loop:**

```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - vim
```

### مقایسه و دیدگاه‌های مختلف

| عنصر | معادل Terraform |
|---|---|
| Inventory | Provider/state (لیست منابع) |
| Playbook | `.tf` files |
| Task+Module | `resource` block |
| Handler | lifecycle hook |
| Idempotency | Idempotency |

**دیدگاه:** Playbook شبیه اسکریپت است اما declarative و idempotent. تفاوت با bash: bash هر بار اجرا می‌شود بدون چک وضعیت؛ Ansible وضعیت را چک می‌کند.

**Tags:** برای اجرای گزینشی taskها: `ansible-playbook --tags "config"`.

### اشتباهات رایج

- **غیرidempotent نوشتن:** مثلاً `command: rm -rf /tmp/x` همیشه "changed" → استفاده از `creates:` یا ماژول مخصوص.
- **فراموش `become: true`:** وظایف مدیریتی شکست می‌خورند.
- **hard-code IP در playbook:** از inventory استفاده کنید.
- **Template بدون `.j2`:** Jinja2 syntax اشتباه.
- **notify بدون handler هم‌نام:** هندلر اجرا نمی‌شود.

### نکات مهم و ترفندها

- **`ansible-lint`:** اعتبارسنجی playbook (مثل yamllint).
- **`--check` (dry run):** اجرای بدون تغییر واقعی (ببینید چه می‌شود).
- **`--diff`:** نمایش تفاوت فایل‌های کپی‌شده.
- **`gather_facts`:** Ansible اطلاعات سیستم (OS، IP) را جمع‌آوری می‌کند (در برخی موارد غیرفعال کنید برای سرعت).
- **`delegate_to`:** اجرای task روی هاست دیگر (مثل اجرا روی localhost برای API call).
- **`serial`:** اجرای تدریجی روی چند هاست (برای zero-downtime).

### خلاصه فصل

اجزای Ansible: Inventory (لیست هاست/گروه)، Playbook (طرح با Play/Task)، Task (واحد کار)، Module (ابزار)، و Handler (واکنش به notify). اجرا با `ansible-playbook`. کلید: نوشتن taskهای idempotent (وضعیت چک شود)، `become: true` برای sudo، استفاده از `template` (Jinja2) به جای copy ساده، و `--check`/`--diff` برای پیش‌نمایش. Tags برای اجرای گزینشی و `serial` برای انتشار بدون وقفه.

---

## فصل ۸۳: Roles و متغیرها (Variables و Vault)

### مدمه

وقتی playbookها بزرگ می‌شوند، نیاز به **ساختاردهی** داریم. **Roles** کد Ansible را ماژولار می‌کنند (مثل ماژول در Terraform)، و **متغیرها/Vault** مدیریت پیکربندی و رمز را ممکن می‌سازند. این فصل سازماندهی حرفه‌ای Ansible را می‌آموزد.

### مفاهیم اساسی

**Role چیست؟**
ساختار پوشه‌بندی استاندارد برای گروه‌بندی tasks، handlers، templates، files و متغیرها مربوط به یک موضوع (مثل `nginx`، `database`).

```
roles/nginx/
├── tasks/main.yml
├── handlers/main.yml
├── templates/nginx.conf.j2
├── files/
├── vars/main.yml
└── defaults/main.yml
```

**استفاده:**
```yaml
roles:
  - nginx
  - database
```

**متغیرها (Variables):**
- **Group/Host vars:** در inventory یا پوشهٔ `group_vars/`، `host_vars/`.
- **Role defaults:** `defaults/main.yml` (کمترین اولویت).
- **Role vars:** `vars/main.yml`.
- **Extra vars:** از CLI (`-e`) — بالاترین اولویت.

**اولویت (Precedence):** extra_vars > play vars > host_vars > group_vars > role defaults.

**Ansible Vault:**
ابزار رمزنگاری فایل‌های متغیر یا کل فایل. رمزها در Git ایمن می‌مانند.

```bash
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-playbook --ask-vault-pass site.yml
```

**جایگزین مدرن:** برای تولید، **Vault با External Secrets** یا ارجاع به AWS Secrets Manager (همانند K8s — فصل ۵۹).

### مثال‌های عملی

**۱. ساختار Role:**

```bash
ansible-galaxy init roles/nginx
```

**۲. `roles/nginx/tasks/main.yml`:**

```yaml
---
- name: Install nginx
  apt:
    name: nginx
    state: present
- name: Template config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx
- name: Start service
  service:
    name: nginx
    state: started
    enabled: true
```

**۳. `roles/nginx/templates/nginx.conf.j2`:**

```jinja
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
    location / {
        proxy_pass http://{{ backend_host }};
    }
}
```

**۴. `group_vars/webservers.yml`:**

```yaml
nginx_port: 80
server_name: example.com
backend_host: 10.0.0.10
```

**۵. Ansible Vault:**

```bash
ansible-vault create group_vars/all/vault.yml
# محتوا:
db_password: supersecret

# در playbook:
- name: Use secret
  debug:
    msg: "{{ db_password }}"
```

### مقایسه و دیدگاه‌های مختلف

| مکانیسم | کاربرد |
|---|---|
| Role | کد ماژولار/قابل استفاده مجدد |
| group_vars/host_vars | پیکربندی محیطی |
| defaults | مقادیر پیش‌فرض role |
| Vault | رمزها |

**دیدگاه:** Roleها استاندارد صنعت برای سازماندهی Ansible هستند (مثل Terraform modules). Ansible Galaxy بازار roleهای آماده است (مثل Terraform Registry).

**Vault محدودیت:** خود Vault فایل را با یک رمز کلیدی رمزنگاری می‌کند — آن رمز باید در CI مدیریت شود (از Vault server یا CI secret استفاده کنید نه hard-code).

### اشتباهات رایج

- **همه چیز در یک playbook صاف:** غیرقابل نگهداری؛ از Role استفاده کنید.
- **رمز در plain vars:** از Vault یا external secret استفاده کنید.
- **اولویت متغیر اشتباه:** فکر می‌کردید مقدار X اعمال شود اما defaults پیروز شد.
- **فراموش `--ask-vault-pass`:** playbook با Vault شکست می‌خورد.
- **Vault password در CI hard-code:** از secret CI استفاده کنید.

### نکات مهم و ترفندها

- **`ansible-galaxy`**: نصب role/collection آماده (`ansible-galaxy install geerlingguy.nginx`).
- **Collections:** بسته‌های ماژول مدرن (مثل `amazon.aws`) — با `requirements.yml`.
- **`ansible-vault encrypt_string`:** رمزنگاری فقط یک مقدار (نه کل فایل).
- **Vault ID:** چند رمز برای محیط‌های مختلف.
- **CI integration:** رمز Vault را از secret CI بخوانید (`--vault-password-file`).
- **Separation:** `defaults` (قابل override) را از `vars` (غیرقابل override توسط user) جدا کنید.

### خلاصه فصل

Roles کد Ansible را ماژولار می‌کنند (مثل Terraform modules) با ساختار پوشهٔ استاندارد (tasks/handlers/templates/vars). متغیرها در group_vars/host_vars/defaults با اولویت مشخص (extra_vars بالاترین) مدیریت می‌شوند. **Ansible Vault** فایل/مقدار را رمزنگاری می‌کند تا رمز در Git ایمن بماند. کلید: استفاده از Role برای سازماندهی، فهم اولویت متغیرها، Vault برای رمز (با رمز در CI از secret)، و استفاده از Collections/ Galaxy برای کد آماده.

---

## فصل ۸۴: Ansible در تیم و ترکیب با Terraform

### مدمه

در تیم‌های واقعی، Ansible باید با سایر ابزارها (مخصوصاً Terraform و CI) همکاری کند. همچنین مدیریت رمز و اجرای خودکار در pipeline ضروری است. این فصل ادغام Ansible در جریان کار تیمی را بررسی می‌کند.

### مفاهیم اساسی

**ترکیب Terraform + Ansible (Terraform Provisioner یا separate):**
دو الگو:
1. **جداگانه (توصیه):** Terraform سرور را می‌سازد، سپس Ansible روی آن پیکربندی می‌کند (با dynamic inventory از خروجی Terraform).
2. **Terraform `remote-exec` provisioner:** Terraform مستقیم Ansible را فراخوانی می‌کند (کمتر تمیز).

**Dynamic Inventory:**
به جای فایل static، Ansible لیست هاست را از API ابر (AWS/GCP) می‌گیرد.

```bash
ansible-inventory -i aws_ec2.yml --list
```

**Ansible در CI:**
- اجرای `ansible-lint` و `ansible-playbook --check` در PR.
- استفاده از Vault password از CI secrets.
- اجرای واقعی پس از approve.

**AWX / Ansible Tower:**
نسخهٔ مدیریت‌شده/وب برای Ansible (schedule، RBAC، UI). متن‌باز: **AWX**؛ تجاری: **Red Hat Ansible Automation Platform**.

**Idempotency در تیم:** همه باید playbookهای idempotent بنویسند تا اجرای مکرر ایمن باشد.

### مثال‌های عملی

**۱. Terraform output → Ansible inventory:**

```hcl
# Terraform
output "web_ips" {
  value = aws_instance.web[*].public_ip
}
```

```bash
# تولید inventory از Terraform output
terraform output -json > tf.json
# یا از dynamic inventory plugin استفاده کنید
```

**۲. Dynamic inventory (AWS):**

```yaml
# aws_ec2.yml
plugin: aws_ec2
regions:
  - us-east-1
keyed_groups:
  - key: tags.Name
    prefix: tag_
```

**۳. CI برای Ansible (GitHub Actions):**

```yaml
- name: Lint
  run: ansible-lint
- name: Check
  run: ansible-playbook -i inventory.yml site.yml --check
- name: Apply
  if: github.ref == 'refs/heads/main'
  run: ansible-playbook -i inventory.yml site.yml
  env:
    VAULT_PASSWORD: ${{ secrets.VAULT_PASSWORD }}
```

**۴. AWX Workflow (مفهومی):**

```
Job Template (playbook) → Schedule (شبانه) → Notification (Slack)
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت |
|---|---|
| Terraform + Ansible جدا | تمیز، تفکیک مسئولیت |
| Terraform provisioner | ساده اما کمتر انعطاف |
| AWX/Tower | مدیریت مرکزی، RBAC |
| Ansible only (بدون TF) | اگر سرورها از قبل هستند |

**دیدگاه:** الگوی استاندارد: **Terraform برای provision، Ansible برای config**. این separation of concerns باعث می‌شود هر ابزار در بهترین کاربردش باشد. AWX برای سازمان‌هایی که نیاز به UI/RBAC/scheduling دارند.

**کانتینرها و K8s:** برای محیط‌های مدرن، بسیاری پیکربندی را به GitOps/K8s منتقل کرده‌اند و Ansible فقط برای بیرون از K8s (مثل VMهای legacy، network devices) استفاده می‌شود.

### اشتباهات رایج

- **استفاده از Terraform provisioner به جای Ansible جدا:** سخت‌تر دیباگ، state پیچیده.
- **Static inventory در ابر:** IPها تغییر می‌کنند → از dynamic استفاده کنید.
- **Vault password در CI hard-code:** از secret استفاده کنید.
- **اجرای apply بدون --check در CI:** تغییر ناخواسته.
- **Ansible روی کانتینرهای K8s:** معمولاً anti-pattern (K8s خودش مدیریت می‌کند).

### نکات مهم و ترفندها

- **`terraform-inventory`** یا pluginها برای bridge بین TF و Ansible.
- **`ansible-pull`:** مدل pull (مثل Puppet) برای scale بزرگ.
- **Collections در `requirements.yml`:** نسخه‌بندی شده.
- **Vault password file در CI:** `echo $VAULT_PASSWORD > .vault_pass && ansible-playbook --vault-password-file=.vault_pass`.
- **Tags + Limits:** اجرای فقط بخشی (`-l webservers --tags nginx`).
- **Molecule:** تست roleهای Ansible (مثل Terraform test).

### خلاصه فصل

ترکیب استاندارد: Terraform provision می‌کند، Ansible پیکربندی می‌کند (جدا، نه با provisioner). Dynamic inventory لیست هاست را از API ابر می‌گیرد (به جای static). در CI: lint + `--check` در PR، apply پس از approve، Vault password از secret. AWX/Tower برای مدیریت مرکزی/RBAC/scheduling. کلید: separation of concerns، dynamic inventory در ابر، و مهاجرت پیکربندی به GitOps/K8s در محیط‌های مدرن (Ansible برای legacy/VM).

---

## فصل ۸۵: بهترین روش‌ها و عیب‌یابی Ansible

### مدمه

Ansible قدرتمند است اما بدون رعایت بهترین روش‌ها، playbookها به سرعت به «اسپاگتی» تبدیل می‌شوند. این فصل چک‌لیست بهترین روش‌ها و متدولوژی عیب‌یابی را جمع‌بندی می‌کند.

### مفاهیم اساسی

**چک‌لیست بهترین روش‌ها:**
1. **Idempotency:** همهٔ taskها باید ایمن برای اجرای مکرر باشند.
2. **Roles:** کد را ماژولار کنید.
3. **Variables در جای درست:** group_vars/host_vars، نه hard-code.
4. **Vault برای رمز:** هرگز رمز plain در Git.
5. **`--check` قبل از اجرا:** ببینید چه تغییری رخ می‌دهد.
6. **`ansible-lint`:** در CI اجباری.
7. **Tags:** برای اجرای گزینشی.
8. **Minimal privilege:** `become` فقط جایی که لازم است.
9. **Version control:** همه در Git.
10. **Collections مدیریت‌شده:** نسخه‌بندی در `requirements.yml`.

**عیب‌یابی لایه‌ای:**
1. **Connectivity:** `ansible all -m ping` (SSH درست است؟).
2. **Inventory صحیح:** `ansible-inventory --list`.
3. **Syntax:** `ansible-playbook --syntax-check`.
4. **Dry run:** `--check --diff`.
5. **Verbose:** `-v` / `-vvv` برای جزئیات SSH.
6. **Module خاص:** با `ansible -m <module>` تست ایزوله.

**مفاهیم کلیدی:**
- **Gathering facts:** اگر کند است و نیاز ندارید، `gather_facts: false`.
- **Serial:** برای آپدیت تدریجی (zero-downtime).
- **`ignore_errors`/`failed_when`:** کنترل شکست.

### مثال‌های عملی

**۱. عیب‌یابی اتصال:**

```bash
ansible all -i inventory.yml -m ping -vvv
# خطای SSH؟ کلید را چک کنید
```

**۲. Dry run:**

```bash
ansible-playbook -i inv.yml site.yml --check --diff
```

**۳. Verbose اجرا:**

```bash
ansible-playbook site.yml -vvv
```

**۴. عیب‌یابی متغیر:**

```yaml
- name: Debug variable
  debug:
    var: nginx_port
```

**۵. تست idempotency:**

```bash
ansible-playbook site.yml   # changed
ansible-playbook site.yml   # باید ok باشد (نه changed)
```

### مقایسه و دیدگاه‌های مختلف

| مشکل | علت رایج | راه حل |
|---|---|---|
| UNREACHABLE | SSH/کلید | بررسی `ansible_user`، کلید |
| PERMISSION | بدون become | اضافه کردن `become: true` |
| changed همیشه | task غیرidempotent | از ماژول مناسب استفاده کنید |
| کندی | gather_facts | خاموش کردن اگر لازم نیست |

**دیدگاه:** فرهنگ «همیشه `--check` قبل از اجرای واقعی روی prod» باید در تیم نهادینه شود. در محیط‌های حساس، حتی `--check` در CI اجباری است.

**Molecule:** برای تست roleها در محیط ایزوله (Docker) پیش از ادغام.

### اشتباهات رایج

- **اجرای مستقیم روی prod بدون --check:** خرابی فاجعه‌بار.
- **playbook غیرidempotent:** rollback غیرممکن.
- **رمز در Git:** نشت اطلاعات.
- **فراموش `gather_facts` برای متغیرهای `ansible_*`**: خطای undefined.
- **SSH با رمز:** کند و ناامن؛ از کلید استفاده کنید.
- **نادیده گرفتن lint:** بدهی فنی انباشته می‌شود.

### نکات مهم و ترفندها

- **`ansible-lint` در pre-commit:** جلوگیری از کد بد.
- **`--limit` برای گروه خاص:** `ansible-playbook -l web1 site.yml`.
- **`serial: 1`** برای آپدیت یکی‌یکی (بدون قطع سرویس).
- **`async`/`poll`** برای taskهای طولانی.
- **Vault in CI:** از secret به عنوان vault password file.
- **Molecule + Docker:** تست خودکار role.

### خلاصه فصل

بهترین روش‌های Ansible: idempotency (اجرای مکرر ایمن)، Roles برای ماژولار بودن، Vault برای رمز، `--check` قبل از اجرا، و `ansible-lint` در CI. عیب‌یابی لایه‌ای: ping (اتصال) → inventory → syntax-check → --check/diff → -vvv. کلید: فرهنگ «همیشه check قبل از prod»، استفاده از SSH keys (نه رمز)، و Molecule برای تست roleها. مشکلات رایج: UNREACHABLE (SSH)، PERMISSION (بدون become)، و عدم idempotency.

---

## منابع و مراجع

- Ansible Documentation: https://docs.ansible.com
- Ansible Best Practices: https://docs.ansible.com/ansible/latest/tips_tricks/index.html
- Ansible Vault: https://docs.ansible.com/ansible/latest/vault_guide/
- Ansible Galaxy: https://galaxy.ansible.com
- AWX (open source Tower): https://github.com/ansible/awx
- Molecule (testing): https://molecule.readthedocs.io
- Terraform + Ansible: https://developer.hashicorp.com/terraform/tutorials/configuration-language/agents
- ansible-lint: https://ansible.readthedocs.io/projects/lint/

**بعدی:** [فایل ۱۸ — مبانی ابر و AWS (Cloud Fundamentals و AWS)](18-Cloud-Fundamentals-and-AWS.md)
