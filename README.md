[main.py](https://github.com/user-attachments/files/32266967/main.py)
import flet as ft
import json
import os
from datetime import datetime, timedelta
import urllib.request
import urllib.error

# رابط قاعدة البيانات السحابية (جاهز للربط الفوري)
CLOUD_URL = "https://agsat-system-default-rtdb.firebaseio.com/data.json"

def load_data_from_cloud():
    try:
        req = urllib.request.Request(CLOUD_URL, headers={"User-Agent": "Mozilla/5.0"})
        with urllib.request.urlopen(req, timeout=5) as response:
            if response.status == 200:
                data = json.loads(response.read().decode("utf-8"))
                if data:
                    return data
    except Exception as e:
        print(f"Cloud load error (fallback to local): {e}")
    
    # في حال عدم توفر إنترنت، يتم الاعتماد على الملف المحلي كبديل مؤقت
        if os.path.exists("database.json"):
            try:
                with open("database.json", "r", encoding="utf-8") as f:
                    return json.load(f)
            except:
                pass
    return {}

def save_data_to_cloud(data):
    try:
        data_json = json.dumps(data, ensure_ascii=False).encode("utf-8")
        req = urllib.request.Request(CLOUD_URL, data=data_json, headers={'Content-Type': 'application/json'}, method='PUT')
        urllib.request.urlopen(req, timeout=5)
    except Exception as e:
        print(f"Cloud save error: {e}")
    
    # حفظ نسخة احتياطية محلية أيضاً في اللابتوب لضمان الأمان
    try:
        with open("database.json", "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=4)
    except Exception as e:
        print(f"Local save error: {e}")

def main(page: ft.Page):
    page.title = "نظام الأقساط وإدارة الأموال (متزامن أونلاين)"
    page.rtl = True
    page.bgcolor = "#0B0F19"
    page.padding = 0
    page.spacing = 0

    app_data = load_data_from_cloud()
    
    credentials = app_data.get("credentials", {"username": "1", "password": "2"})
    customers_data = app_data.get("customers", {})

    def save_app_state():
        full_data = {
            "credentials": credentials,
            "customers": customers_data
        }
        save_data_to_cloud(full_data)

    # --- شاشة تسجيل الدخول ---
    def show_login_screen():
        page.clean()
        page.bgcolor = "#0B192C"
        page.vertical_alignment = ft.MainAxisAlignment.CENTER
        page.horizontal_alignment = ft.CrossAxisAlignment.CENTER

        custom_logo = ft.Container(
            content=ft.Row([
                ft.Text("H S", size=22, weight=ft.FontWeight.BOLD, color="#38bdf8"),
                ft.VerticalDivider(width=10, color="#38bdf8", thickness=2),
                ft.Column([
                    ft.Text("SYSTEMS", size=11, weight=ft.FontWeight.BOLD, color="white"),
                    ft.Text("FINANCE & INSTALLMENTS", size=8, color="#94A3B8"),
                ], spacing=0)
            ], alignment=ft.MainAxisAlignment.CENTER, spacing=10),
            padding=12,
            bgcolor="#1E3E62",
            border_radius=8
        )

        error_text = ft.Text(value="", color="#f87171", weight=ft.FontWeight.BOLD, size=12)

        def handle_login(e):
            if username_field.value == credentials["username"] and password_field.value == credentials["password"]:
                show_main_app()
            else:
                error_text.value = "خطأ في اسم المستخدم أو كلمة السر!"
                page.update()

        def handle_cancel(e):
            username_field.value = ""
            password_field.value = ""
            error_text.value = ""
            page.update()

        username_field = ft.TextField(
            label="اسم المستخدم",
            value="",
            color="black",
            bgcolor="white",
            border_color="black",
            height=52,
            text_size=15
        )
        
        password_field = ft.TextField(
            label="كلمة السر",
            value="",
            password=True,
            can_reveal_password=True,
            color="black",
            bgcolor="white",
            border_color="black",
            height=52,
            text_size=15,
            on_submit=handle_login
        )

        login_box = ft.Container(
            content=ft.Column([
                username_field,
                password_field,
                error_text,
                ft.Divider(height=5, color="transparent"),
                ft.Row([
                    ft.GestureDetector(
                        on_tap=handle_login,
                        content=ft.Container(
                            content=ft.Text("موافق", color="black", weight=ft.FontWeight.BOLD, text_align=ft.TextAlign.CENTER),
                            bgcolor="white",
                            padding=12,
                            border_radius=6,
                            expand=True
                        )
                    ),
                    ft.GestureDetector(
                        on_tap=handle_cancel,
                        content=ft.Container(
                            content=ft.Text("إلغاء الأمر", color="black", weight=ft.FontWeight.BOLD, text_align=ft.TextAlign.CENTER),
                            bgcolor="white",
                            padding=12,
                            border_radius=6,
                            expand=True
                        )
                    ),
                ], alignment=ft.MainAxisAlignment.CENTER, spacing=10)
            ], spacing=15, horizontal_alignment=ft.CrossAxisAlignment.CENTER),
            padding=25,
            bgcolor="#1E3E62",
            width=420,
            border_radius=12
        )

        header_section = ft.Column([
            custom_logo,
            ft.Divider(height=10, color="transparent"),
            ft.Text("نظام لإدارة الحسابات والمبيعات", size=14, color="#94A3B8", weight=ft.FontWeight.BOLD),
            ft.Divider(height=5, color="transparent"),
            ft.Text("نظام الأقساط وإدارة الأموال (أونلاين)", size=24, color="white", weight=ft.FontWeight.BOLD),
            ft.Divider(height=10, color="white", thickness=1),
        ], horizontal_alignment=ft.CrossAxisAlignment.CENTER)

        footer_section = ft.Column([
            ft.Text("نظام لإدارة الحسابات والمبيعات", size=11, color="white"),
            ft.Text("مبرمج حسين ثائر", size=13, color="#38bdf8", weight=ft.FontWeight.BOLD),
            ft.Text("Mob: 07752168736", size=12, color="white")
        ], horizontal_alignment=ft.CrossAxisAlignment.START, spacing=3)

        page.add(
            ft.Column([
                header_section,
                ft.Divider(height=15, color="transparent"),
                ft.Row([
                    footer_section,
                    login_box
                ], alignment=ft.MainAxisAlignment.SPACE_AROUND, vertical_alignment=ft.CrossAxisAlignment.CENTER)
            ], alignment=ft.MainAxisAlignment.CENTER, horizontal_alignment=ft.CrossAxisAlignment.CENTER)
        )
        page.update()

    # --- النظام الرئيسي بعد تسجيل الدخول ---
    def show_main_app():
        page.clean()
        page.bgcolor = "#0B0F19"
        page.vertical_alignment = ft.VerticalAlignment.START
        page.horizontal_alignment = ft.CrossAxisAlignment.START

        content_area = ft.Container(expand=True, padding=30)

        def change_view(content_widget):
            content_area.content = content_widget
            page.update()

        def show_dashboard(e=None):
            # تحديث البيانات من السحابة عند كل زيارة للوحة التحكم لضمان المزامنة
            nonlocal customers_data
            latest_data = load_data_from_cloud()
            if "customers" in latest_data:
                customers_data.update(latest_data["customers"])

            total_cust = len(customers_data)
            total_remaining = sum(c.get("remaining", 0) for c in customers_data.values())
            total_profit = sum(c.get("profit", c.get("total", 0) - c.get("cost", 0)) for c in customers_data.values())

            dashboard_view = ft.Column([
                ft.Row([
                    ft.Text("📊 نظرة عامة على النظام (متزامن أونلاين)", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                    ft.Row([
                        ft.GestureDetector(
                            on_tap=show_change_credentials_screen,
                            content=ft.Container(
                                content=ft.Row([
                                    ft.Text("🔐", size=15),
                                    ft.Text("تغيير معلومات الدخول", color="white", weight=ft.FontWeight.BOLD, size=13)
                                ], spacing=8),
                                bgcolor="#1F2937", padding=10, border_radius=8,
                                border=ft.Border(top=ft.BorderSide(1, "#374151"), bottom=ft.BorderSide(1, "#374151"), left=ft.BorderSide(1, "#374151"), right=ft.BorderSide(1, "#374151"))
                            )
                        ),
                        ft.GestureDetector(
                            on_tap=lambda e: show_login_screen(),
                            content=ft.Container(
                                content=ft.Row([
                                    ft.Text("🚪", size=15),
                                    ft.Text("تسجيل خروج", color="white", weight=ft.FontWeight.BOLD, size=13)
                                ], spacing=8),
                                bgcolor="#7F1D1D", padding=10, border_radius=8,
                                border=ft.Border(top=ft.BorderSide(1, "#991B1B"), bottom=ft.BorderSide(1, "#991B1B"), left=ft.BorderSide(1, "#991B1B"), right=ft.BorderSide(1, "#991B1B"))
                            )
                        )
                    ], spacing=10)
                ], alignment=ft.MainAxisAlignment.SPACE_BETWEEN),
                ft.Divider(height=15, color="transparent"),
                ft.Row([
                    ft.Container(
                        content=ft.Column([
                            ft.Text("إجمالي الزبائن", size=14, color="#94A3B8"),
                            ft.Text(str(total_cust), size=28, weight=ft.FontWeight.BOLD, color="#38BDF8")
                        ], spacing=5),
                        bgcolor="#111827", padding=20, border_radius=12, expand=True,
                        border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937"))
                    ),
                    ft.Container(
                        content=ft.Column([
                            ft.Text("إجمالي الديون المتبقية", size=14, color="#94A3B8"),
                            ft.Text(f"{total_remaining:,} د.ع", size=28, weight=ft.FontWeight.BOLD, color="#F43F5E")
                        ], spacing=5),
                        bgcolor="#111827", padding=20, border_radius=12, expand=True,
                        border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937"))
                    ),
                    ft.Container(
                        content=ft.Column([
                            ft.Text("إجمالي الأرباح المتوقعة", size=14, color="#94A3B8"),
                            ft.Text(f"{total_profit:,} د.ع", size=28, weight=ft.FontWeight.BOLD, color="#10B981")
                        ], spacing=5),
                        bgcolor="#111827", padding=20, border_radius=12, expand=True,
                        border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937"))
                    ),
                ], spacing=20),
                ft.Divider(height=20, color="transparent"),
                ft.Container(
                    content=ft.Column([
                        ft.Text("مرحباً بك في نظام إدارة الأقساط السحابي", size=20, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                        ft.Text("تم ربط النظام أونلاين بنجاح، أي تحديث يتم إجراؤه سيتم مزامنته فوراً.", size=15, color="#94A3B8"),
                    ], spacing=10),
                    bgcolor="#111827", padding=25, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937"))
                ),
                ft.Divider(height=10, color="transparent"),
                ft.Container(
                    content=ft.Row([
                        ft.Text("💻 تطوير المبرمج: حسين ثائر", color="#38BDF8", weight=ft.FontWeight.BOLD, size=15),
                        ft.Text("📞 07752168736", color="#E2E8F0", weight=ft.FontWeight.BOLD, size=15),
                    ], alignment=ft.MainAxisAlignment.SPACE_BETWEEN),
                    bgcolor="#111827", padding=20, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#2563EB"), bottom=ft.BorderSide(1, "#2563EB"), left=ft.BorderSide(1, "#2563EB"), right=ft.BorderSide(1, "#2563EB"))
                )
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(dashboard_view)

        def show_change_credentials_screen(e=None):
            new_user_field = ft.TextField(label="اسم المستخدم الجديد", value=credentials["username"], color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            new_pass_field = ft.TextField(label="كلمة السر الجديدة", value=credentials["password"], color="white", bgcolor="#1F2937", height=55, text_size=15, password=True, can_reveal_password=True, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            msg_txt = ft.Text(value="", size=14, weight=ft.FontWeight.BOLD)

            def save_new_creds(e):
                if new_user_field.value and new_pass_field.value:
                    credentials["username"] = new_user_field.value
                    credentials["password"] = new_pass_field.value
                    save_app_state()
                    msg_txt.value = "✨ تم تحديث بيانات تسجيل الدخول ورفعها للسحابة بنجاح!"
                    msg_txt.color = "#10B981"
                else:
                    msg_txt.value = "⚠️ يرجى عدم ترك الحقول فارغة!"
                    msg_txt.color = "#F43F5E"
                page.update()

            view = ft.Column([
                ft.Text("🔐 تغيير معلومات تسجيل الدخول", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                ft.Divider(height=10, color="transparent"),
                ft.Container(
                    content=ft.Column([
                        new_user_field,
                        new_pass_field,
                        msg_txt,
                        ft.Divider(height=10, color="transparent"),
                        ft.Row([
                            ft.GestureDetector(
                                on_tap=save_new_creds,
                                content=ft.Container(
                                    content=ft.Text("💾 حفظ البيانات وتحديث السحابة", color="white", weight=ft.FontWeight.BOLD, size=15, text_align=ft.TextAlign.CENTER),
                                    bgcolor="#2563EB", padding=15, border_radius=10, width=240
                                )
                            )
                        ], alignment=ft.MainAxisAlignment.START)
                    ], spacing=20),
                    bgcolor="#111827", padding=35, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937")), width=500
                )
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(view)

        def show_add_customer_screen(e=None):
            name_field = ft.TextField(label="اسم الزبون الكامل", color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            phone_field = ft.TextField(label="رقم الهاتف", color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            device_field = ft.TextField(label="نوع المادة أو الجهاز", color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            price_field = ft.TextField(label="سعر المنتج (البيع)", color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            cost_field = ft.TextField(label="رأس المال (التكلفة)", color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            advance_field = ft.TextField(label="المقدمة المدفوعة", color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            
            system_dropdown = ft.Dropdown(
                label="نظام القسط",
                label_style=ft.TextStyle(color="#94A3B8", size=14),
                color="white",
                bgcolor="#1F2937",
                height=55,
                text_size=15,
                value="شهري",
                border_color="#374151",
                focused_border_color="#38BDF8",
                options=[
                    ft.dropdown.Option("شهري"),
                    ft.dropdown.Option("أسبوعي"),
                    ft.dropdown.Option("يومي"),
                ]
            )

            msg_text = ft.Text(value="", size=14, weight=ft.FontWeight.BOLD)

            def save_customer(e):
                if name_field.value and phone_field.value and price_field.value and cost_field.value:
                    try:
                        total_price = float(price_field.value)
                        cost = float(cost_field.value)
                        advance = float(advance_field.value) if advance_field.value else 0
                    except ValueError:
                        msg_text.value = "يرجى إدخال أرقام صحيحة في الحقول المالية!"
                        msg_text.color = "#F43F5E"
                        page.update()
                        return

                    remaining = total_price - advance
                    profit = total_price - cost
                    name = name_field.value
                    system_type = system_dropdown.value

                    now = datetime.now()
                    if system_type == "شهري":
                        next_due = now + timedelta(days=30)
                    elif system_type == "أسبوعي":
                        next_due = now + timedelta(days=7)
                    else:
                        next_due = now + timedelta(days=1)

                    current_date_str = now.strftime("%Y-%m-%d %H:%M")
                    next_due_str = next_due.strftime("%Y-%m-%d")

                    customers_data[name] = {
                        "phone": phone_field.value,
                        "device": device_field.value if device_field.value else "جهاز عام",
                        "total": total_price,
                        "cost": cost,
                        "profit": profit,
                        "paid": advance,
                        "remaining": remaining,
                        "system_type": system_type,
                        "next_due_date": next_due_str,
                        "history": [f"إضافة الحساب بقيمة {total_price} ومقدمة {advance} بنظام ({system_type}) بتاريخ {current_date_str}"]
                    }

                    save_app_state()
                    msg_text.value = f"✨ تم حفظ الزبون {name} ورفع البيانات للسحابة! القسط القادم: {next_due_str}"
                    msg_text.color = "#10B981"
                    name_field.value = ""
                    phone_field.value = ""
                    device_field.value = ""
                    price_field.value = ""
                    cost_field.value = ""
                    advance_field.value = ""
                    page.update()
                else:
                    msg_text.value = "⚠️ يرجى ملء كافة الحقول الأساسية المطلوبة!"
                    msg_text.color = "#F43F5E"
                    page.update()

            form_view = ft.Column([
                ft.Text("➕ إضافة زبون جديد للحسابات", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                ft.Divider(height=10, color="transparent"),
                ft.Container(
                    content=ft.Column([
                        ft.Row([ft.Container(content=name_field, expand=True), ft.Container(content=phone_field, expand=True)], spacing=20),
                        ft.Row([ft.Container(content=device_field, expand=True), ft.Container(content=system_dropdown, expand=True)], spacing=20),
                        ft.Row([ft.Container(content=price_field, expand=True), ft.Container(content=cost_field, expand=True), ft.Container(content=advance_field, expand=True)], spacing=20),
                        msg_text,
                        ft.Divider(height=10, color="transparent"),
                        ft.Row([
                            ft.GestureDetector(
                                on_tap=save_customer,
                                content=ft.Container(
                                    content=ft.Text("💾 حفظ الزبون ومزامنته سحابياً", color="white", weight=ft.FontWeight.BOLD, size=15, text_align=ft.TextAlign.CENTER),
                                    bgcolor="#2563EB", padding=15, border_radius=10, width=250
                                )
                            )
                        ], alignment=ft.MainAxisAlignment.START)
                    ], spacing=20),
                    bgcolor="#111827", padding=35, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937"))
                )
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(form_view)

        def show_customers_count_screen(e=None):
            def delete_customer(name):
                if name in customers_data:
                    del customers_data[name]
                    save_app_state()
                    show_customers_count_screen()

            cards = []
            for name, data in customers_data.items():
                remaining_val = data.get('remaining', 0)
                phone = data.get('phone', 'غير متوفر')
                device = data.get('device', 'غير محدد')
                
                card = ft.Container(
                    content=ft.Row([
                        ft.Column([
                            ft.Text(f"👤 {name}", color="white", weight=ft.FontWeight.BOLD, size=16),
                            ft.Text(f"📞 هاتف: {phone} | 📦 الجهاز: {device}", color="#94A3B8", size=13),
                        ], spacing=4, expand=True),
                        ft.Column([
                            ft.Text(f"المتبقي: {remaining_val:,} د.ع", color="#F43F5E", weight=ft.FontWeight.BOLD, size=15),
                        ]),
                        ft.Container(width=20),
                        ft.GestureDetector(
                            on_tap=lambda e, n=name: delete_customer(n),
                            content=ft.Container(
                                content=ft.Text("🗑️ حذف", color="white", weight=ft.FontWeight.BOLD, size=13),
                                bgcolor="#DC2626", padding=10, border_radius=8
                            )
                        )
                    ], alignment=ft.MainAxisAlignment.SPACE_BETWEEN),
                    bgcolor="#1F2937", padding=15, border_radius=10,
                    border=ft.Border(top=ft.BorderSide(1, "#374151"), bottom=ft.BorderSide(1, "#374151"), left=ft.BorderSide(1, "#374151"), right=ft.BorderSide(1, "#374151"))
                )
                cards.append(card)

            if not cards:
                cards.append(ft.Text("لا يوجد زبائن مسجلين في النظام حالياً.", color="#94A3B8", size=15))

            view = ft.Column([
                ft.Row([
                    ft.Text("👥 إدارة وحذف الزبائن", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                    ft.Container(
                        content=ft.Text(f"العدد الكلي: {len(customers_data)}", color="#38BDF8", weight=ft.FontWeight.BOLD, size=14),
                        bgcolor="#1F2937", padding=10, border_radius=8
                    )
                ], alignment=ft.MainAxisAlignment.SPACE_BETWEEN),
                ft.Divider(height=10, color="transparent"),
                ft.Container(
                    content=ft.Column(cards, spacing=12, scroll=ft.ScrollMode.AUTO),
                    bgcolor="#111827", padding=25, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937")), height=500
                )
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(view)

        def show_pay_debt_screen(e=None):
            search_field = ft.TextField(label="اسم الزبون", color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            amount_field = ft.TextField(label="مبلغ التسديد المدفوع", color="white", bgcolor="#1F2937", height=55, text_size=15, label_style=ft.TextStyle(color="#94A3B8", size=14), border_color="#374151", focused_border_color="#38BDF8")
            res_text = ft.Text(value="", size=14, weight=ft.FontWeight.BOLD)

            def submit_payment(e):
                name = search_field.value
                if name in customers_data and amount_field.value:
                    try:
                        paid_amount = float(amount_field.value)
                    except ValueError:
                        res_text.value = "⚠️ يرجى إدخال مبلغ صحيح!"
                        res_text.color = "#F43F5E"
                        page.update()
                        return

                    cust = customers_data[name]
                    cust["paid"] += paid_amount
                    cust["remaining"] -= paid_amount

                    now = datetime.now()
                    current_date = now.strftime("%Y-%m-%d %H:%M")
                    
                    system_type = cust.get("system_type", "شهري")
                    if system_type == "شهري":
                        next_due = now + timedelta(days=30)
                    elif system_type == "أسبوعي":
                        next_due = now + timedelta(days=7)
                    else:
                        next_due = now + timedelta(days=1)
                    
                    cust["next_due_date"] = next_due.strftime("%Y-%m-%d")

                    if "history" not in cust:
                        cust["history"] = []
                    cust["history"].append(f"تسديد مبلغ {paid_amount} د.ع وتحديث موعد القسط إلى {cust['next_due_date']} بتاريخ {current_date}")

                    save_app_state()
                    show_receipt_screen(name, paid_amount, cust["remaining"], current_date, cust["device"])
                else:
                    res_text.value = "⚠️ الزبون غير موجود أو لم تقم بإدخال المبلغ!"
                    res_text.color = "#F43F5E"
                    page.update()

            view = ft.Column([
                ft.Text("💵 تسديد ديون وإصدار وصل", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                ft.Divider(height=10, color="transparent"),
                ft.Container(
                    content=ft.Column([
                        search_field,
                        amount_field,
                        res_text,
                        ft.Divider(height=10, color="transparent"),
                        ft.Row([
                            ft.GestureDetector(
                                on_tap=submit_payment,
                                content=ft.Container(
                                    content=ft.Text("🖨️ إتمام التسديد وتحديث السحابة", color="white", weight=ft.FontWeight.BOLD, size=15, text_align=ft.TextAlign.CENTER),
                                    bgcolor="#10B981", padding=15, border_radius=10, width=260
                                )
                            )
                        ], alignment=ft.MainAxisAlignment.START)
                    ], spacing=20),
                    bgcolor="#111827", padding=35, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937"))
                )
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(view)

        def show_receipt_screen(customer_name, paid_amt, remaining_amt, payment_date, device_type):
            print_msg = ft.Text(value="", size=14, weight=ft.FontWeight.BOLD)

            def print_receipt_action(e):
                html_content = f"""
                <!DOCTYPE html>
                <html lang="ar" dir="rtl">
                <head>
                    <meta charset="UTF-8">
                    <title>سند قبض</title>
                    <style>
                        body {{ font-family: 'Tahoma', Arial, sans-serif; background: #fff; color: #000; padding: 20px; }}
                        .receipt-box {{ border: 2px solid #b45309; padding: 25px; border-radius: 12px; max-width: 480px; margin: auto; background: #fffdf5; }}
                        .header {{ text-align: center; border-bottom: 2px dashed #b45309; padding-bottom: 12px; margin-bottom: 15px; }}
                        .header h2 {{ color: #92400e; margin: 0; font-size: 24px; }}
                        .header p {{ color: #78350f; margin: 5px 0 0; font-size: 15px; font-weight: bold; }}
                        .row {{ display: flex; justify-content: space-between; margin-bottom: 12px; font-size: 16px; }}
                        .footer {{ text-align: center; margin-top: 25px; border-top: 2px dashed #b45309; padding-top: 12px; font-size: 13px; color: #555; }}
                    </style>
                </head>
                <body onload="window.print()">
                    <div class="receipt-box">
                        <div class="header">
                            <h2>سند قبض</h2>
                            <p>مكتب المهندس للأقساط</p>
                        </div>
                        <div class="row"><span><b>التاريخ والوقت:</b></span> <span>{payment_date}</span></div>
                        <div class="row"><span><b>إستلمنا من السيد/ة:</b></span> <span>{customer_name}</span></div>
                        <div class="row"><span><b>مبلغ وقدره فقط:</b></span> <span style="color: #047857; font-weight: bold;">{paid_amt:,} د.ع</span></div>
                        <div class="row"><span><b>وذلك مقابل:</b></span> <span>تسديد قسط عن ({device_type})</span></div>
                        <div class="row"><span><b>المبلغ المتبقي:</b></span> <span style="color: #b91c1c; font-weight: bold;">{remaining_amt:,} د.ع</span></div>
                        <div class="footer">
                            <p><b>المحاسب:</b> ترف | <b>المبرمج:</b> حسين ثائر</p>
                            <p>لا يعتبر هذا السند نهائياً إلا بعد تحصيل المبلغ.</p>
                        </div>
                    </div>
                </body>
                </html>
                """
                try:
                    import tempfile
                    tf = tempfile.NamedTemporaryFile(delete=False, suffix=".html", mode="w", encoding="utf-8")
                    tf.write(html_content)
                    tf.close()
                    os.startfile(tf.name)
                    print_msg.value = "🖨️ تم فتح نافذة الطباعة بنجاح!"
                    print_msg.color = "#10B981"
                except Exception as ex:
                    print_msg.value = f"خطأ: {str(ex)}"
                    print_msg.color = "#F43F5E"
                page.update()

            view = ft.Column([
                ft.Text("🧾 معاينة وإصدار سند القبض", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                ft.Divider(height=10, color="transparent"),
                ft.Container(
                    content=ft.Column([
                        ft.Text("🏢 مكتب المهندس للأقساط", color="#FBBF24", size=18, weight=ft.FontWeight.BOLD),
                        ft.Divider(color="#374151"),
                        ft.Text(f"👤 اسم الزبون: {customer_name}", color="white", size=16),
                        ft.Text(f"💵 المبلغ المدفوع: {paid_amt:,} د.ع", color="#10B981", size=16, weight=ft.FontWeight.BOLD),
                        ft.Text(f"⚠️ المبلغ المتبقي: {remaining_amt:,} د.ع", color="#F43F5E", size=16, weight=ft.FontWeight.BOLD),
                        ft.Text(f"📦 مقابل: {device_type}", color="#94A3B8", size=15),
                        ft.Text(f"📅 التاريخ: {payment_date}", color="#94A3B8", size=15),
                        print_msg,
                        ft.Divider(color="#374151"),
                        ft.Row([
                            ft.GestureDetector(
                                on_tap=print_receipt_action,
                                content=ft.Container(content=ft.Text("🖨️ طباعة الوصل الآن", color="white", weight=ft.FontWeight.BOLD, size=15, text_align=ft.TextAlign.CENTER), bgcolor="#3B82F6", padding=15, border_radius=10, expand=True)
                            ),
                            ft.GestureDetector(
                                on_tap=show_pay_debt_screen,
                                content=ft.Container(content=ft.Text("تسديد جديد", color="white", weight=ft.FontWeight.BOLD, size=15, text_align=ft.TextAlign.CENTER), bgcolor="#374151", padding=15, border_radius=10, expand=True)
                            )
                        ], spacing=15)
                    ], spacing=15),
                    bgcolor="#111827", padding=35, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937")), width=600
                )
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(view)

        def show_late_customers_screen(e=None):
            today_str = datetime.now().strftime("%Y-%m-%d")
            today_date = datetime.now().date()
            
            late_cards = []
            for name, data in customers_data.items():
                remaining_val = data.get('remaining', 0)
                next_due_date_str = data.get('next_due_date', today_str)
                
                try:
                    due_date = datetime.strptime(next_due_date_str, "%Y-%m-%d").date()
                except:
                    due_date = today_date

                if remaining_val > 0 and due_date <= today_date:
                    phone = data.get('phone', 'غير متوفر')
                    system_type = data.get('system_type', 'شهري')
                    card = ft.Container(
                        content=ft.Row([
                            ft.Column([
                                ft.Text(f"👤 {name}", color="white", weight=ft.FontWeight.BOLD, size=16),
                                ft.Text(f"📞 هاتف: {phone} | ⏳ النظام: {system_type}", color="#94A3B8", size=13),
                                ft.Text(f"📅 موعد الاستحقاق: {next_due_date_str}", color="#FBBF24", size=13, weight=ft.FontWeight.BOLD),
                            ], spacing=4, expand=True),
                            ft.Text(f"المتبقي: {remaining_val:,} د.ع", color="#F43F5E", weight=ft.FontWeight.BOLD, size=16)
                        ], alignment=ft.MainAxisAlignment.SPACE_BETWEEN),
                        bgcolor="#1F2937", padding=15, border_radius=10,
                        border=ft.Border(top=ft.BorderSide(1, "#374151"), bottom=ft.BorderSide(1, "#374151"), left=ft.BorderSide(1, "#374151"), right=ft.BorderSide(1, "#374151"))
                    )
                    late_cards.append(card)

            if not late_cards:
                late_cards.append(ft.Text("لا يوجد زبائن متأخرون عن موعد السداد حالياً 🎉", color="#10B981", size=16, weight=ft.FontWeight.BOLD))

            view = ft.Column([
                ft.Text("⚠️ الزبائن المتأخرون عن الدفع", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                ft.Divider(height=10, color="transparent"),
                ft.Container(
                    content=ft.Column(late_cards, spacing=12, scroll=ft.ScrollMode.AUTO),
                    bgcolor="#111827", padding=25, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937")), height=500
                )
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(view)

        def show_statistics_screen(e=None):
            total_capital = sum(float(c.get("cost", 0)) for c in customers_data.values())
            total_profit = sum(float(c.get("profit", c.get("total", 0) - c.get("cost", 0))) for c in customers_data.values())
            
            cards = []
            for name, data in customers_data.items():
                cost = data.get("cost", 0)
                profit = data.get("profit", data.get("total", 0) - cost)
                remaining = data.get("remaining", 0)
                
                card = ft.Container(
                    content=ft.Row([
                        ft.Text(f"👤 {name}", color="white", weight=ft.FontWeight.BOLD, size=15, expand=True),
                        ft.Text(f"رأس المال: {cost:,} د.ع", color="#38BDF8", size=13, expand=True),
                        ft.Text(f"الربح: {profit:,} د.ع", color="#10B981", size=13, expand=True),
                        ft.Text(f"المتبقي: {remaining:,} د.ع", color="#F43F5E", size=13, weight=ft.FontWeight.BOLD, expand=True),
                    ], alignment=ft.MainAxisAlignment.SPACE_BETWEEN),
                    bgcolor="#1F2937", padding=15, border_radius=10,
                    border=ft.Border(top=ft.BorderSide(1, "#374151"), bottom=ft.BorderSide(1, "#374151"), left=ft.BorderSide(1, "#374151"), right=ft.BorderSide(1, "#374151"))
                )
                cards.append(card)

            view = ft.Column([
                ft.Text("📊 إحصائيات الأرباح ورأس المال", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                ft.Text(f"إجمالي رأس المال: {total_capital:,} د.ع | إجمالي الأرباح المتوقعة: {total_profit:,} د.ع", color="#38BDF8", size=15, weight=ft.FontWeight.BOLD),
                ft.Divider(height=10, color="transparent"),
                ft.Container(
                    content=ft.Column(cards, spacing=12, scroll=ft.ScrollMode.AUTO),
                    bgcolor="#111827", padding=25, border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937")), height=460
                )
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(view)

        def show_statement_screen(e=None):
            cust_search = ft.TextField(
                label="أدخل اسم الزبون للبحث وعرض كشف الحساب",
                color="white",
                bgcolor="#1F2937",
                height=55,
                text_size=15,
                label_style=ft.TextStyle(color="#94A3B8", size=14),
                border_color="#374151",
                focused_border_color="#38BDF8"
            )
            result_container = ft.Column([], spacing=10)

            def search_customer(e):
                result_container.controls.clear()
                name = cust_search.value
                if name in customers_data:
                    cust = customers_data[name]
                    raw_history = cust.get("history", [])
                    history_items = [ft.Text(f"• {h}", color="#94A3B8", size=13) for h in reversed(raw_history)]

                    statement_box = ft.Container(
                        content=ft.Column([
                            ft.Text(f"👤 اسم الزبون: {name}", color="white", weight=ft.FontWeight.BOLD, size=18),
                            ft.Text(f"📞 رقم الهاتف: {cust['phone']} | 📦 الجهاز: {cust['device']}", color="#94A3B8", size=15),
                            ft.Text(f"⏳ نظام القسط: {cust.get('system_type', 'شهري')} | 📅 القسط القادم: {cust.get('next_due_date', 'غير محدد')}", color="#FBBF24", size=15, weight=ft.FontWeight.BOLD),
                            ft.Divider(color="#374151"),
                            ft.Row([
                                ft.Text(f"💰 السعر الكلي: {cust['total']:,} د.ع", color="#38BDF8", size=15),
                                ft.Text(f"💵 المدفوع: {cust['paid']:,} د.ع", color="#10B981", size=15),
                                ft.Text(f"⚠️ المتبقي: {cust['remaining']:,} د.ع", color="#F43F5E", size=15, weight=ft.FontWeight.BOLD),
                            ], alignment=ft.MainAxisAlignment.SPACE_BETWEEN),
                            ft.Divider(color="#374151"),
                            ft.Text("📅 سجل العمليات والدفعات السابقة:", color="#38BDF8", weight=ft.FontWeight.BOLD, size=15),
                            ft.Container(
                                content=ft.Column(history_items, spacing=8, scroll=ft.ScrollMode.AUTO),
                                height=200
                            )
                        ], spacing=12),
                        bgcolor="#111827", padding=25, border_radius=12,
                        border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937")),
                        width=700
                    )
                    result_container.controls.append(statement_box)
                else:
                    result_container.controls.append(ft.Text("⚠️ الزبون غير موجود في النظام!", color="#F43F5E", size=15, weight=ft.FontWeight.BOLD))
                page.update()

            view = ft.Column([
                ft.Text("📑 كشف حساب زبون", size=24, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                ft.Divider(height=10, color="transparent"),
                ft.Row([
                    ft.Container(content=cust_search, width=450),
                    ft.GestureDetector(
                        on_tap=search_customer,
                        content=ft.Container(
                            content=ft.Text("بحث", color="white", weight=ft.FontWeight.BOLD, size=15, text_align=ft.TextAlign.CENTER),
                            bgcolor="#2563EB", padding=16, border_radius=10, width=130
                        )
                    )
                ], spacing=15),
                ft.Divider(height=10, color="transparent"),
                result_container
            ], spacing=10, scroll=ft.ScrollMode.AUTO)
            change_view(view)

        def create_nav_btn(text, action, icon_symbol):
            return ft.GestureDetector(
                on_tap=action,
                content=ft.Container(
                    content=ft.Row([
                        ft.Text(icon_symbol, size=18),
                        ft.Text(text, color="#E2E8F0", weight=ft.FontWeight.BOLD, size=15),
                    ], spacing=15),
                    bgcolor="#111827",
                    padding=16,
                    border_radius=12,
                    border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937"))
                )
            )

        programmer_info = ft.Container(
            content=ft.Column([
                ft.Divider(color="#1F2937", height=10),
                ft.Text("💻 تطوير المبرمج:", size=12, color="#94A3B8"),
                ft.Text("حسين ثائر", size=14, weight=ft.FontWeight.BOLD, color="#38BDF8"),
                ft.Text("📞 07752168736", size=12, color="#E2E8F0"),
            ], spacing=4),
            padding=10,
            bgcolor="#111827",
            border_radius=10,
            border=ft.Border(top=ft.BorderSide(1, "#1F2937"), bottom=ft.BorderSide(1, "#1F2937"), left=ft.BorderSide(1, "#1F2937"), right=ft.BorderSide(1, "#1F2937"))
        )

        sidebar = ft.Container(
            content=ft.Column([
                ft.Container(
                    content=ft.Column([
                        ft.Text("⚡ مكتب المهندس", size=20, weight=ft.FontWeight.BOLD, color="#F8FAFC"),
                        ft.Text("نظام الأقساط (سحابي)", size=12, color="#94A3B8"),
                    ], spacing=4),
                    padding=10
                ),
                ft.Divider(color="#1F2937", height=20),
                create_nav_btn("لوحة التحكم الرئيسية", show_dashboard, "🏠"),
                create_nav_btn("إضافة زبون جديد", show_add_customer_screen, "➕"),
                create_nav_btn("إدارة وحذف الزبائن", show_customers_count_screen, "👥"),
                create_nav_btn("تسديد ديون وإصدار وصل", show_pay_debt_screen, "💵"),
                create_nav_btn("الزبائن المتأخرون", show_late_customers_screen, "⚠️"),
                create_nav_btn("إحصائيات الأرباح", show_statistics_screen, "📊"),
                create_nav_btn("كشف حساب زبون", show_statement_screen, "📑"),
                ft.Container(expand=True),
                programmer_info
            ], spacing=10, scroll=ft.ScrollMode.AUTO),
            bgcolor="#0D1322",
            padding=25,
            width=300,
            border=ft.Border(left=ft.BorderSide(1, "#1F2937"))
        )

        main_layout = ft.Row([
            sidebar,
            content_area
        ], expand=True, spacing=0)

        page.add(main_layout)
        show_dashboard()

    show_login_screen()

ft.app(target=main)
