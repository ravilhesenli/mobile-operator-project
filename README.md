# 📱 Mobile Operator CRM & Billing Database Project

Bu layihə, bir Mobil Operator şirkətinin abonentlərini, tarif planlarını, daxili/beynəlxalq zəng və SMS tarixçələrini, maliyyə ödənişlərini, cihaz məlumatlarını (IMEI/OS) və müştəri dəstək sorğularını (CRM) idarə etmək üçün nəzərdə tutulmuş **Oracle SQL verilənlər bazası modelidir**.

Layihə sənaye standartlarına uyğun qurulmuş 14 əsas cədvəldən, real test məlumatlarından və biznes analitikası (Business Intelligence) üçün xüsusi olaraq hazırlanmış 15 kompleks SQL sorğusundan ibarətdir.

---

## 📊 Verilənlər Bazası Arxitekturası (Schema)
Layihədəki bütün cədvəllər Primary Key və Foreign Key constraint-ləri vasitəsilə bir-biri ilə sıx əlaqələndirilmişdir:
- **Abonent & Cihaz Bloku:** `subscribers`, `address`, `address_type`, `technical_data`
- **Tarif & Xidmətlər:** `tariff_informations`, `services`
- **İstifadə Tarixçəsi (Traffic):** `calls`, `call_type`, `sms_informations`, `sms_types`
- **Maliyyə & Faturalandırma:** `payments`, `payment_method_type`
- **CRM & Dəstək:** `complaints_and_support_requests`, `complaint_type`

---

## 🚀 Önə Çıxan Analitik Sorğular (Business Analytics)
Biznes idarəçiliyi və marketinq qərarlarının qəbul edilməsi üçün hazırlanmış 15 sorğudan bəzi mühüm nümunələr:

### 1. Müştəri Seqmentasiyası (Yaş və Cinsiyyət Bölgüsü)
Mobil operatorun hədəf kütləsini analiz etmək üçün abonentləri yaş aralıqlarına (`18 ve aşağı`, `19-30`, `31-50`, `51 ve yuxarı`) və cinsiyyətinə görə qruplaşdırır:
```sql
select gender, count(*) as say,
case when trunc((sysdate-birth_date)/365)<=18 then '18 ve asagi'
     when trunc((sysdate-birth_date)/365) between 19 and 30 then '19-30'
     when trunc((sysdate-birth_date)/365) between 31 and 50 then '31-50'
     else '51 ve yuxari'
end as yas_qruplari
from subscribers
group by gender, ...
```

### 2. CRM Analitikası (Ən Çox Şikayət Edən TOP 10 Abonent)
Xidmət keyfiyyətini artırmaq məqsədilə sistemdə ən çox şikayət və ya texniki problem bildirişi yaradan ilk 10 müştərini sıralayır (Sorğuda Subquery, JOIN və WITH variantları optimallaşdırılmışdır):
```sql
select s.name, s.surname, count(csr.description) as say
from subscribers s inner join complaints_and_support_requests csr
on s.subscribers_id=csr.subscribers_id 
where lower(csr.description) like lower('%issue%')
group by s.name, s.surname 
order by say desc
fetch next 10 rows only;
```

### 3. Maliyyə Risklərinin Təyini (Aktiv Olmayan Ödəyicilər)
Statusu `PASSIVE` (müvəqqəti qapadılmış və s.) olan, lakin son 6 ay ərzində sistemdə ödəniş etmiş abonentləri tapır. Bu data marketinq komandası üçün müştərini geri qazanma (Churn Winback) kampaniyalarında kritik rol oynayır.
## 🛠️ Necə İstifadə Etməli?
1. `schema.sql` faylındakı kodları Oracle SQL Developer və ya PL/SQL Developer mühitində işə salaraq cədvəlləri qurun və test məlumatlarını (`INSERT`) daxil edin.
2. `queries.sql` faylındakı 15 analitik sorğunu icra edərək idarəetmə hesabatlarını (Dashboard data) əldə edin.
