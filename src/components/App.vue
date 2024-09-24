<template>
    <div id="app">
        <div class="card" ref="cardfrontside" style="position: relative;">
            <h1 style="position: absolute; top: 10mm; left: 5mm; width: 50mm; font:bold;">
                <b>
                    {{ name }}
                </b>
            </h1>
            <p style="position: absolute; top: 20mm; left: 5mm; width: 50mm;">{{ title }}</p>
            <p style="position: absolute; top: 35mm; left: 8mm;">{{ phone }}</p>
            <p style="position: absolute; top: 39mm; left: 8mm;">{{ phonecell }}</p>
            <p style="position: absolute; bottom: 3mm; left: 48mm;" v-html="addrforcard"></p>
            <!-- <img src="@/assets/logo.png" width="155mm" style="position: absolute; top: 3mm; left: 40mm;"> -->
            <img alt="Здесь может быть ваш логотип..." width="155mm" style="position: absolute; top: 3mm; left: 40mm;">
            <div id="bar" ref="bar"></div>
            <div id="barleft"></div>

            <p style="position: absolute; bottom: 3mm; left: 8mm; font-size: 8pt;" v-html="email" ref="textemail"></p>


        </div>
        <div>


        </div>
        <div class="cardbackside" ref="cardbackside">

            <img :src="qrCodeImage" alt="QR Code IMAGE" style="width: 48mm; height: 48mm;"
                title="например: Иван Иванович Иванов" />

        </div>

        <div class="form" style="margin-left: 10pt;">
            <label>ФИО:</label>
            <input v-model="name" @input="generateQRCode" placeholder="Имя, Отчество, Фамилия" v-clearonesc
                title="например: Иван Иванович Иванов" />
            <label>Должность:</label>
            <input v-model="title" @input="generateQRCode" placeholder="Должность" v-clearonesc
                title="например: Главный специалист по......." />
            <label>E-mail:</label>
            <input v-model="email" @input="generateQRCode" placeholder="Email " v-clearonesc @keydown="adjustFontSize"
                title="например: i.ivanov@mogilev.belapb.by" />
            <label>Телефон сотовый:</label>
            <MaskInput v-model="phonecell" mask="+375 (##) ###-##-###" placeholder="+375 (29) --- -- --" v-clearonesc
                title="Вводить только цифры " />
            <label>Телефон рабочий:</label>
            <!-- <input v-model="phone" @input="generateQRCode" placeholder="Телефон с кодом" /> -->
            <MaskInput v-model="phone" mask="+375 (###) ##-##-##" placeholder="+375 (222) -- -- --" v-clearonesc
                title="Вводить только цифры " />
            <label>Полный адрес:</label>
            <input v-model="address" @touchstart="generateQRCode" placeholder="Республика Беларусь" readonly
                title="Выберите область">
            <!-- <input v-model="region" @touchstart="generateQRCode" placeholder="......ская область" /> -->
            <select v-model="region">
                <option v-for="option in regionoptions" :key="option.value" :value="option.value">
                    {{ option.text }}
                </option>
            </select>
            <input v-model="city" @touchstart="generateQRCode" placeholder="город, городской поселок, деревня и т.д."
                v-clearonesc title="например: г. Калинковичи" :readonly="region === 'г. Минск'" />
            <input v-model="street" @touchstart="generateQRCode" placeholder="улица, проспект и т.д." v-clearonesc
                title="например: ул. Социалистическая" />
            <input v-model="housenumber" @touchstart="generateQRCode" placeholder="номер дома, корпус" v-clearonesc
                title="например: д.124" />

            <MaskInput v-model="zipcode" mask="######" placeholder="почтовый индекс" v-clearonesc
                title="например: 220221" />
            <button @click="createPDF">Сохранить pdf для последущей печати</button>
        </div>

        <footer role="contentinfo">
            <span style="font-size: 12px;">
                (C)
            </span>
            <a href="mailto:a.kovalenko@belapb.by&?subject=QRcode.Vcard ">8_kovalenko_351</a>
        </footer>

    </div>
    <footer>
        * Цвет и логотип можно установить любые по вашим пожеланиям...
    </footer>
</template>

<script>
import { ref, watchEffect, onMounted, onUnmounted } from 'vue'
import html2canvas from 'html2canvas';
import QRCode from 'qrcode'
import jsPDF from 'jspdf';
import { MaskInput } from 'vue-3-mask';
// import html2pdf from 'html2pdf.js';

export default {
    components: { MaskInput, },
    setup() {
        const qrCodeImage = ref('')
        const data = ref('');
        const name = ref('');
        const title = ref('');
        const email = ref('');
        const phone = ref('');
        const phonecell = ref('');
        const address = ref("Республика Беларусь");
        const region = ref('');
        const city = ref('');
        const street = ref('');
        const housenumber = ref('');
        const zipcode = ref('');

        const addrforcard = ref('');
        const addrforcontact = ref('');
        const cardfrontside = ref(null);
        const cardbackside = ref(null);
        const parent = ref(null);
        const textemail = ref(null);
        const bar = ref(null);

        const isOverlap = (element1, element2) => {
            var rect1 = element1.value.getBoundingClientRect();
            var rect2 = element2.value.getBoundingClientRect();

            var overlap = !(rect1.right < rect2.left ||
                rect1.left > rect2.right ||
                rect1.bottom < rect2.top ||
                rect1.top > rect2.bottom)
            return overlap;
        }
        const adjustFontSize = () => {
            const ptKoeff = 0.74999943307122;
            let fontSize = parseFloat(textemail.value.style.fontSize) * ptKoeff;
            // console.log(textemail.value.getBoundingClientRect());
            let textemailrect = textemail.value.getBoundingClientRect();
            const barrect = bar.value.getBoundingClientRect();

            if (isOverlap(textemail, bar)) {
                //     // if (textemailrect.x + textemailrect.width + 5 > barrect.x) {
                while (textemailrect.x + textemailrect.width + 1 > barrect.x) {
                    fontSize -= 0.2;
                    // console.log(fontSize);
                    textemail.value.style.fontSize = fontSize + "pt";
                    // textemail.value.style.fontSize = "7.5pt"
                    textemailrect = textemail.value.getBoundingClientRect();
                    // fontSize = parseFloat(window.getComputedStyle(textemail.value).fontSize);
                }
            } else {
                do {
                    fontSize += 0.1;
                    // console.log(fontSize);
                    textemail.value.style.fontSize = Math.min(fontSize, 8) + "pt";
                    textemailrect = textemail.value.getBoundingClientRect();
                    if (textemailrect.x + textemailrect.width + 2 >= barrect.x) {
                        fontSize -= 0.1;
                        textemail.value.style.fontSize = fontSize + "pt";
                        // textemailrect = textemail.value.getBoundingClientRect();
                        break;
                    }
                } while (textemailrect.x + textemailrect.width - 2 <= barrect.x && fontSize <= 8)
            }

        };
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
        const clearInput = async () => {
            this.inputValue = '';
        }

        watchEffect(async () => {
            const n = ref(name.value.split(" ").join(";"));
            const revision = ref(getCurrentDateTimeInVCardFormat());
            const tmpaddrforcontact = ref(`ADR:;;${street.value}, ${housenumber.value};${city.value};${region.value};${zipcode.value};${address.value}`);
            addrforcontact.value = tmpaddrforcontact.value;
            const textValue = ref(`BEGIN:VCARD\nVERSION:3.0\nN:${n.value};;\nFN:${name.value}\nORG:ОАО "Белагропромбанк"\nTITLE:${title.value}\nEMAIL;TYPE=work:${email.value}
            \nTEL;TYPE=work,voice;VALUE=uri:tel:${phone.value}\nTEL;TYPE=cell,voice;VALUE=uri:tel:${phonecell.value}\n${addrforcontact.value}\nREV:${revision.value}\nEND:VCARD`);

            data.value = textValue.value;
            // console.log(data.value);
            // console.log(addrforcontact.value);
            //ADR:;;пр.Мира, 55;г.Могилев;Могилевская область;212027;Республика Беларусь
            // addrforcard.value = address.value + ",\n" + region.value.trim().toString() + ',\n' + city.value.trim() + ', ' + street.value.trim() + ', ' + housenumber.value.trim();
            addrforcard.value = address.value + ",<br>";
            (region.value.trim() == 'г. Минск') ? addrforcard.value = addrforcard.value +
                region.value.trim() + ',<br>' + street.value.trim() + ',' + housenumber.value.trim()
                : addrforcard.value = addrforcard.value +
                region.value.trim()
                + ',<br>' + city.value.trim() + ',<br>' + street.value.trim() + ',' + housenumber.value.trim();
            ;
            // addrforcard.value = addrforcard.value + city.value.trim() + ', ' + street.value.trim() + ', ' + housenumber.value.trim();

            //  if (generateQRCode != null) 
            await generateQRCode();
            adjustFontSize();

        });
        const generateQRCode = async () => {
            try {
                var segs = [
                    { data: data.value.toString(), mode: 'byte' },

                ]

                qrCodeImage.value = await QRCode.toDataURL(segs, {
                    errorCorrectionLevel: 'L',

                })
            } catch (err) {
                console.error(err)
            }
        }

        // onBeforeMount(generateQRCode);
        onMounted(() => {
            window.addEventListener('resize', adjustFontSize);
            generateQRCode();
            // adjustFontSize();
        });

        onUnmounted(() => {
            window.removeEventListener('resize', adjustFontSize);
        });

        // Создание PDF из HTML-контента
        const createPDF = async () => {
            // const quality = 0.9; // Установите желаемое качество (от 0 до 1)
            const img1 = await html2canvas(cardbackside.value, { scale: 10 });
            const img2 = await html2canvas(cardfrontside.value, { scale: 10 });
            //надо попробовать использовать
            // const imgData = img1.toDataURL('image/png', quality);
            // const imgData2 = img2.toDataURL('image/png', quality);

            const doc = new jsPDF({
                unit: 'mm',
                userUnit: 300, // Установите DPI здесь. В вебе используется 72, но вы можете изменить на 150 или 300
                marginLeft: 15,
                marginRight: 15,
                marginTop: 0,
                marginBottom: 0,
                precision: 4,
                orientation: 'p',
            });

            const cardWidth = 90;
            const cardHeight = 50;
            const margin = 15;
            const verticalSpace = 0;
            const horizontalSpace = 0;

            const cardsInRow = Math.floor((doc.internal.pageSize.width - 2 * margin + horizontalSpace) / (cardWidth + horizontalSpace));
            const cardsInColumn = Math.floor((doc.internal.pageSize.height - 2 * margin + verticalSpace) / (cardHeight + verticalSpace));

            for (let i = 0; i < cardsInColumn; i++) {
                for (let j = 0; j < cardsInRow; j++) {
                    const x = margin + j * (cardWidth + horizontalSpace);
                    const y = margin + i * (cardHeight + verticalSpace);
                    doc.addImage(img1, 'JPEG', x + horizontalSpace, y + verticalSpace, cardWidth, cardHeight);
                }
            }

            doc.addPage();
            doc.setPage(2);

            for (let i = 0; i < cardsInColumn; i++) {
                for (let j = 0; j < cardsInRow; j++) {
                    const x = margin + j * (cardWidth + horizontalSpace);
                    const y = margin + i * (cardHeight + verticalSpace);
                    doc.addImage(img2, 'JPEG', x + horizontalSpace, y + verticalSpace, cardWidth, cardHeight);
                }
            }

            doc.save('Визитные карточки ' + name.value + '.pdf');
        }


        return {
            bar,
            textemail,
            parent,
            clearInput,
            qrCodeImage, MaskInput,
            data,
            name,
            title,
            email,
            phone,
            phonecell,
            address,
            region,
            city,
            street,
            housenumber,
            zipcode,
            addrforcard,
            addrforcontact,
            cardfrontside,
            cardbackside,

            createPDF,


            regionoptions: [
                { text: 'г. Минск', value: 'г. Минск' },
                { text: 'Брестская область', value: 'Брестская область' },
                { text: 'Витебская область', value: 'Витебская область' },
                { text: 'Гомельская область', value: 'Гомельская область' },
                { text: 'Гродненская область', value: 'Гродненская область' },
                { text: 'Минская область', value: 'Минская область' },
                { text: 'Могилевская область', value: 'Могилевская область' },
                // { text: 'г. Минск ', value: 'г. Минск' }
            ]
        }
    },

    directives: {
        // enables v-focus in template
        clearonesc: {
            mounted(el) {
                el.addEventListener('keydown', async function (event) {
                    if (event.key === 'Escape') {
                        el.value = "";
                        // el = ref(null);
                    }
                });
            }
        }
    }
}
</script>

<style>
label {
    color: black;
}

p {
    color: black;
}

b {
    color: black;
}

h1 {
    font-family: 'Circe Bold';
    font-size: 11pt;
}

/* body {
    background-color: white;
    color: black;
} */
body {
    margin: 0;
    /* color: #050505; */
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

/* header {
     background: blue; *
}*/

main {
    /* Чтобы занимал оставшееся пространство */
    flex-grow: 1;
    /* background: red; */
}

footer {
    /* Чтобы footer не уменьшался */
    flex-shrink: 0;
    background: rgb(245, 242, 242);
    text-align: right;
    font-size: 12px;
    position: absolute;
    bottom: 0;
    width: 100%;
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
    border: 0.5px;
    border-style: dotted;
    border-color: rgb(121, 115, 115);
    padding: 5mm;
    margin: 5mm;
}

.cardbackside {
    width: 90mm;
    height: 50mm;
    /* border: .5px dotted rgb(121, 115, 115); */

    display: flex;
    justify-content: center;
    align-items: center;

}

.form input {
    display: block;
    margin-bottom: 5px;
    width: 450px;
}

#bar {
    width: 1mm;
    height: 13mm;
    bottom: 3mm;
    background-color: rgb(69, 38, 241);
    position: absolute;
    left: 45mm;
}

#barleft {
    width: 1mm;
    height: 13mm;
    bottom: 3mm;
    background-color: rgb(69, 38, 241);
    position: absolute;
    left: 5mm;
}

@font-face {
    font-family: 'Circe Bold';
    src: url('@/assets/circe-bold.ttf') format('truetype');
    /* путь к файлу шрифта */
}

body {
    font-family: 'Circe Light';
    font-size: 9pt;
    line-height: 9.5pt;

}

h1 {
    font-family: 'Circe Bold';
    font-size: 11pt;
    line-height: 13pt;
}

select {
    margin-bottom: 5pt;
}


@font-face {
    font-family: 'Circe Light';
    src: url('@/assets/circe_light.ttf') format('truetype');
    /* путь к файлу шрифта */
}

/* Стили для темной темы */
@media (prefers-color-scheme: dark) {
    /* Ваши стили для темной темы */
}
</style>
