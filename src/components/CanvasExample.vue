<template>
    <div id="app">



        <div class="card" ref="card" style="position: relative;">
            <h1 style="position: absolute; top: 10mm; left: 5mm; width: 50mm; font:bold;">
                <b>
                    {{ name }}
                </b>
            </h1>
            <p style="position: absolute; top: 20mm; left: 5mm; width: 50mm;">{{ title }}</p>
            <p style="position: absolute; top: 33mm; left: 5mm;">{{ phone }}</p>
            <p style="position: absolute; top: 36.5mm; left: 5mm;">{{ phonecell }}</p>
            <p style="position: absolute; bottom: 5mm; left: 48mm;">{{ address }}</p>
            <img src="@/assets/колосок_белагропромбанк.png" width="155mm"
                style="position: absolute; top: 3mm; left: 40mm;">
            <div id="bar"></div>
            <div id="barleft"></div>
            <p style="position: absolute; bottom: 5mm; left: 5mm;">{{ email }}</p>


            <div class="entry-content" style="position: inherit; top: 50mm ; left:55mm; z-index: -1;">
                <QRCodeVue3 :width="200" :height="200" :value="data" :key="data"
                    :qr-options="{ errorCorrectionLevel: 'M' }"
                    :image-options="{ hideBackgroundDots: true, imageSize: 0.1, margin: 5 }"
                    :corners-square-options="{ type: 'dot', color: 'black' }"
                    :corners-dot-options="{ type: undefined, color: 'black' }" :dots-options="{
                        type: 'dots',
                        color: 'black'
                    }" :download="false" />


            </div>
        </div>



        <div class="form">
            <label>ФИО:</label>
            <input v-model="name" placeholder="Имя" />
            <label>Должность:</label>
            <input v-model="title" placeholder="Должность" />
            <label>E-mail:</label>
            <input v-model="email" placeholder="Email" />
            <label>Телефон сотовый:</label>
            <input v-model="phonecell" placeholder="Телефон" />
            <label>Телефон рабочий:</label>
            <input v-model="phone" placeholder="Телефон с кодом" />
            <label>Полный адрес:</label>
            <input v-model="address" placeholder="Республика Беларусь, Могилевская область, пр. Мира, 55" />

            <button @click="saveCard">Сохранить</button>
        </div>

    </div>
</template>

<script>
import { ref, watchEffect } from 'vue';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';
import QRCodeVue3 from 'qrcode-vue3'
export default {
    components: { QRCodeVue3 },

    setup() {
        const data = ref('');
        const name = ref('Иванов Иван Иваныч');
        const title = ref('главный специалист по ничегонеделанию');
        const email = ref('i.ivanov@belapb.b');
        const phone = ref('+375 17 123 54 65');
        const address = ref("Республика Беларусь, Могилевская область, г.Могилев, пр.Мира, 55");
        const phonecell = ref('+375 29 123-45-66');
        const card = ref(null);


        function getCurrentDateTimeInVCardFormat() {
            const date = new Date();
            const year = date.getUTCFullYear();
            const month = String(date.getUTCMonth() + 1).padStart(2, '0');
            const day = String(date.getUTCDate()).padStart(2, '0');
            const hours = String(date.getUTCHours()).padStart(2, '0');
            const minutes = String(date.getUTCMinutes()).padStart(2, '0');
            const seconds = String(date.getUTCSeconds()).padStart(2, '0');

            return `${year}${month}${day}T${hours}${minutes}${seconds}Z`;
        }


        watchEffect(() => {
            const n = ref(name.value.split(" ").join(";"));
            /* 
BEGIN:VCARD
VERSION:4.0
N:Иванов;Иван;Иванович;;
FN:Иван Иванович Иванов
ORG:ООО "Рога и Копыта"
TITLE:Генеральный директор
EMAIL;TYPE=work:ivanov@example.com
TEL;TYPE=work,voice;VALUE=uri:tel:+7-495-123-4567
TEL;TYPE=home,voice;VALUE=uri:tel:+7-495-765-4321
ADR;TYPE=work;LABEL="г. Москва, ул. Пушкина, д. Колотушкина, 1":;;ул. Пушкина, д. Колотушкина, 1;Москва;;123456;Россия
ADR;TYPE=home;LABEL="г. Москва, ул. Лермонтова, д. 10":;;ул. Лермонтова, д. 10;Москва;;654321;Россия
REV:20240601T195243Z
END:VCARD

 */


            // const revision = ref(getCurrentDateTimeInVCardFormat());
            // const textValue = ref(`BEGIN:VCARD\nVERSION:4.0\nN:${n.value};;\nFN:${name.value}\nORG:ОАО "Белагропромбанк"\nTITLE:${title.value}\nEMAIL;TYPE=work:${email.value}\nTEL;TYPE=work,voice;VALUE=uri:tel:${phone.value}\nTEL;TYPE=cell,voice;VALUE=uri:tel:${phonecell.value}\nADR;TYPE=work;LABEL="${address.value}":;;;;;;\nREV:${revision.value}\nEND:VCARD`);
            // //    let data = 'Текст для преобразования в BLOB';
            // let blob = new Blob([new Uint8Array([0xEF, 0xBB, 0xBF]), textValue.value], { type: 'text/plain;charset=utf-8' });
            // let url = URL.createObjectURL(blob);


            /*
            
            BEGIN:VCARD
            VERSION:3.0
            N:Gump;Forrest;;Mr.;
            FN:Forrest Gump
            ORG:Bubba Gump Shrimp Co.
            TITLE:Shrimp Man
            TEL;TYPE=WORK,VOICE:(111) 555-1212
            EMAIL:forrestgump@example.com
            END:VCARD
            
            
            
            */
            const revision = ref(getCurrentDateTimeInVCardFormat());
            const textValue = ref(`BEGIN:VCARD\nVERSION:3.0\nN:${n.value};;\nFN:${name.value}\nORG:'ОАО "Белагропромбанк"'\nTITLE:${title.value}\nEMAIL;TYPE=work:${email.value}\nTEL;TYPE=work,voice;VALUE=uri:tel:${phone.value}\nTEL;TYPE=cell,voice;VALUE=uri:tel:${phonecell.value}\nADR;TYPE=work;LABEL="${address.value}":;;;;;;\nREV:${revision.value}\nEND:VCARD`);

            data.value = textValue.value;
            console.log(data.value);
        });
        const saveCard = () => {
            html2canvas(card.value).then(canvas => {
                const imgData = canvas.toDataURL('image/png');
                // const pdf = new jsPDF();
                // const doc = new jsPDF('p', 'mm', 'a4'); // Создаем новый документ jsPDF в портретной ориентации, миллиметрах и формате A4
                let doc = new jsPDF({
                    orientation: 'portrait',
                    unit: 'mm',
                    format: [297, 210], // размер страницы A4
                    putOnlyUsedFonts: true,
                    floatPrecision: 16 // или "smart", значение по умолчанию, для оптимальной точности
                });

                // Увеличиваем масштаб
                doc.scale(4.0, 4.0);
                const cardWidth = 90; // Ширина визитной карточки в мм
                const cardHeight = 50; // Высота визитной карточки в мм
                const margin = 5; // Отступ от края страницы до визитной карточки в мм
                const verticalSpace = 0; // Вертикальное пространство между визитными карточками в мм
                const horizontalSpace = 0; // Горизонтальное пространство между визитными карточками в мм

                const cardsInRow = Math.floor((doc.internal.pageSize.width - 2 * margin + horizontalSpace) / (cardWidth + horizontalSpace)); // Количество визитных карточек в ряду
                const cardsInColumn = Math.floor((doc.internal.pageSize.height - 2 * margin + verticalSpace) / (cardHeight + verticalSpace)); // Количество визитных карточек в столбце
                for (let i = 0; i < cardsInColumn; i++) {
                    for (let j = 0; j < cardsInRow; j++) {
                        const x = margin + j * (cardWidth + horizontalSpace); // x-координата верхнего левого угла визитной карточки
                        const y = margin + i * (cardHeight + verticalSpace); // y-координата верхнего левого угла визитной карточки

                        // Здесь вы можете добавить свой код для создания визитной карточки
                        // Например, добавим простой текст в каждую визитную карточку
                        // doc.text('Визитная карточка ' + (i * cardsInRow + j + 1), x + 5, y + 25); // Добавляем текст в центр визитной карточки
                        doc.addImage(imgData, 'PNG', x + 5, y + 5, cardWidth, cardHeight);
                    }
                }

                doc.save("download.pdf");
            });
        };

        return {
            data,
            name,
            title,
            email,
            phone,
            phonecell,
            card,
            address,
            saveCard
        };
    }
};
</script>

<style>
h1 {
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-size: 11pt;
}

body {
    background-color: white;
    color: black;
}

#app {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

.card {
    width: 90mm;
    height: 50mm;
    border: 1px dotted rgb(0, 0, 0);
    padding: 5mm;
    margin: 5mm;
}

.form input {
    display: block;
    margin-bottom: 5px;
    width: 300px;
}

#bar {
    width: 1mm;
    height: 13mm;
    bottom: 3mm;
    background-color: #f15a26;
    position: absolute;
    left: 45mm;
}

#barleft {
    width: 1mm;
    height: 13mm;
    bottom: 3mm;
    background-color: #f15a26;
    position: absolute;
    left: 2mm;
}

@font-face {
    font-family: 'Circe Bold';
    src: url('Circe-Bold.ttf') format('truetype');
    /* путь к файлу шрифта */
}

body {
    font-family: 'Circe Bold', sans-serif;
    font-size: 9pt;
    line-height: 9.5pt;

}

h1 {
    font-family: 'Circe Light', sans-serif;
    font-size: 11pt;
    line-height: 13pt;
}

@font-face {
    font-family: 'Circe Light';
    src: url('Circe-Light.ttf') format('truetype');
    /* путь к файлу шрифта */
}
</style>