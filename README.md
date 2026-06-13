[property.html](https://github.com/user-attachments/files/28912493/property.html)
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تفاصيل العقار - إرث العقارية</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: system-ui, 'Cairo', sans-serif; }
        body { background: #f0f4f8; padding: 20px; }
        .container { max-width: 900px; margin: auto; background: white; border-radius: 32px; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.1); }
        .loading, .error { text-align: center; padding: 60px 20px; font-size: 1.2rem; color: #555; }
        .error { color: #c00; }
        .gallery { display: flex; overflow-x: auto; gap: 8px; padding: 12px; background: #1e2a3a; }
        .gallery img { max-height: 300px; border-radius: 20px; flex-shrink: 0; }
        .info { padding: 24px; }
        .price { font-size: 2rem; font-weight: bold; color: #006b5e; background: #e0f2e9; display: inline-block; padding: 8px 20px; border-radius: 40px; margin-bottom: 20px; }
        h1 { font-size: 1.8rem; margin-bottom: 12px; }
        .location { color: #2c5f5a; margin-bottom: 20px; }
        .detail-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px,1fr)); gap: 12px; margin: 20px 0; }
        .detail-card { background: #f8fafc; border-radius: 20px; padding: 12px; text-align: center; }
        .desc { background: #f9fbfd; padding: 20px; border-radius: 24px; line-height: 1.6; margin: 20px 0; }
        .btn { display: block; background: #006b5e; color: white; text-align: center; padding: 14px; border-radius: 40px; text-decoration: none; font-weight: bold; margin-top: 20px; }
        .btn:hover { background: #004d44; }
        footer { text-align: center; padding: 16px; color: #777; font-size: 0.8rem; }
    </style>
</head>
<body>
<div class="container" id="content">
    <div class="loading">⏳ جاري تحميل بيانات العقار...</div>
</div>

<script>
    // 🔧 إعدادات Airtable الخاصة بك
    const AIRTABLE_BASE_ID = 'appwY0M6F2lzjYt8J';     // Base ID (بدون أي جزء إضافي)
    const AIRTABLE_TABLE_NAME = 'العقارات';           // اسم جدول العقارات
    const AIRTABLE_API_KEY = 'patliYzgTSzMBhu87.9a0eb39b72d02f35598f6155bfe1d9ba1c0fbe0b2f6e86c86d7dbb233361c103';  // التوكن الكامل

    // قراءة كود العقار من الرابط
    const urlParams = new URLSearchParams(window.location.search);
    const propertyCode = urlParams.get('code');

    if (!propertyCode) {
        document.getElementById('content').innerHTML = `<div class="error">❌ لم يتم إرسال كود العقار. الرجاء استخدام رابط صحيح (مثل ?code=PR-2).</div>`;
    } else {
        fetchProperty(propertyCode);
    }

    async function fetchProperty(code) {
        const filterFormula = `{كود العقار الداخلي} = "${code}"`;
        const url = `https://api.airtable.com/v0/${AIRTABLE_BASE_ID}/${encodeURIComponent(AIRTABLE_TABLE_NAME)}?filterByFormula=${encodeURIComponent(filterFormula)}&maxRecords=1`;

        try {
            const response = await fetch(url, {
                headers: { 'Authorization': `Bearer ${AIRTABLE_API_KEY}` }
            });
            const data = await response.json();

            if (!response.ok) {
                throw new Error(`خطأ في الاستجابة: ${response.status}`);
            }

            if (!data.records || data.records.length === 0) {
                document.getElementById('content').innerHTML = `<div class="error">⚠️ لم نجد عقاراً بهذا الكود: ${code}</div>`;
                return;
            }

            const fields = data.records[0].fields;
            renderProperty(fields);
        } catch (err) {
            console.error(err);
            document.getElementById('content').innerHTML = `<div class="error">🚫 خطأ في الاتصال بقاعدة البيانات. تأكد من إعدادات API وأن الكود صحيح.</div>`;
        }
    }

    function renderProperty(fields) {
        const name = fields['اسم العقار'] || 'بدون عنوان';
        const type = fields['نوع العقار'] || 'عقار';
        const location = fields['الموقع (المدينة)'] || '';
        const area = fields['المساحة (م²)'] ? fields['المساحة (م²)'] + ' م²' : 'غير محدد';
        let price = fields['السعر المطلوب الحالي'];
        price = price ? Number(price).toLocaleString() + ' ريال' : 'السعر غير محدد حالياً';
        const desc = fields['وصف تفصيلي'] || fields['وصف مختصر'] || 'لا يوجد وصف إضافي.';
        const mapLink = fields['رابط الموقع (خرائط)'] || '';

        // صور
        let imagesHtml = '';
        if (fields['صور العقار'] && Array.isArray(fields['صور العقار']) && fields['صور العقار'].length) {
            imagesHtml = `<div class="gallery">${fields['صور العقار'].map(img => `<img src="${img.url}" alt="صورة العقار">`).join('')}</div>`;
        }

        const fullHtml = `
            ${imagesHtml}
            <div class="info">
                <div class="price">💰 ${price}</div>
                <h1>${name}</h1>
                <div class="location">📍 ${location}</div>
                <div class="detail-grid">
                    <div class="detail-card">🏷️ نوع العقار<br><strong>${type}</strong></div>
                    <div class="detail-card">📐 المساحة<br><strong>${area}</strong></div>
                </div>
                <div class="desc">📄 ${desc}</div>
                ${mapLink ? `<div style="margin:12px 0"><a href="${mapLink}" target="_blank">🗺️ عرض الموقع على الخريطة</a></div>` : ''}
                <a href="tel:783171777" class="btn">📞 اتصل بنا للاستفسار (783171777)</a>
                <div style="margin-top:16px; text-align:center; color:#aaa; font-size:0.75rem;">كود العقار: ${propertyCode}</div>
            </div>
        `;
        document.getElementById('content').innerHTML = fullHtml;
    }
</script>
</body>
</html>
