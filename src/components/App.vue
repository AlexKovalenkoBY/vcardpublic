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
            <img :src="imageSrc" alt="Логотип" ref="logo" :style="imageStyle" @mousedown="startDrag"
                @touchstart="startDrag">
            <div id="bar" ref="bar"></div>
            <div id="barleft"></div>
            <p style="position: absolute; bottom: 3mm; left: 8mm; font-size: 8pt;" v-html="email" ref="textemail"></p>
        </div>
        <div class="cardbackside" ref="cardbackside">
            <img :src="qrCodeImage" alt="QR Code IMAGE" style="width: 48mm; height: 48mm;" />
        </div>
        <div class="form" style="margin-left: 10pt;">
            <label>ФИО:</label>
            <input v-model="name" @input="generateQRCode" placeholder="Имя, Отчество, Фамилия" v-clearonesc />
            <label>Должность:</label>
            <input v-model="title" @input="generateQRCode" placeholder="Должность" v-clearonesc />
            <label>E-mail:</label>
            <input v-model="email" @input="generateQRCode" placeholder="Email " v-clearonesc
                @keydown="adjustFontSize" />
            <label>Телефон сотовый:</label>
            <MaskInput v-model="phonecell" mask="+### (##) ###-##-###" placeholder="+### (29) --- -- --" v-clearonesc />
            <label>Телефон рабочий:</label>
            <MaskInput v-model="phone" mask="+### (###) ##-##-##" placeholder="+### (222) -- -- --" v-clearonesc />
            <label>Полный адрес:</label>
            <input v-model="address" @touchstart="generateQRCode" placeholder="Российская федерация" readonly />
            <select v-model="region">
                <option v-for="option in regionoptions" :key="option.value" :value="option.value">
                    {{ option.text }}
                </option>
            </select>
            <input v-model="city" @touchstart="generateQRCode" placeholder="город, городской поселок, деревня и т.д."
                v-clearonesc :readonly="region === 'г. Москва'" />
            <input v-model="street" @touchstart="generateQRCode" placeholder="улица, проспект и т.д." v-clearonesc />
            <input v-model="housenumber" @touchstart="generateQRCode" placeholder="номер дома, корпус" v-clearonesc />
            <MaskInput v-model="zipcode" mask="######" placeholder="почтовый индекс" v-clearonesc />
            <button @click="createPDF">Сохранить pdf для последущей печати</button>
        </div>
        <footer role="contentinfo">
            <span style="font-size: 12px;">(C)</span>
            <a href="mailto:a.kovalenko@belapb.by&?subject=QRcode.Vcard ">8_kovalenko_351</a>
        </footer>
    </div>
    <footer>* Цвет и логотип можно установить любые по вашим пожеланиям...</footer>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'
import html2canvas from 'html2canvas';
import QRCode from 'qrcode'
import jsPDF from 'jspdf';
import { MaskInput } from 'vue-3-mask';

export default {
    components: { MaskInput },
    setup() {
        const qrCodeImage = ref('');
        const data = ref('');
        const name = ref('');
        const title = ref('');
        const email = ref('');
        const phone = ref('');
        const phonecell = ref('');
        const address = ref("Российская Федерация");
        const region = ref('');
        const city = ref('');
        const street = ref('');
        const housenumber = ref('');
        const zipcode = ref('');
        const addrforcard = ref('');
        const addrforcontact = ref('');
        const cardfrontside = ref(null);
        const cardbackside = ref(null);
        const textemail = ref(null);
        const bar = ref(null);
        const imageSrc = ref('');
        const imageStyle = ref({
            position: 'absolute',
            top: '3mm',
            left: '60mm',
            width: '65mm',
            cursor: 'move',
        });
        const isDragging = ref(false);
        const startX = ref(0);
        const startY = ref(0);
        const startLeft = ref(0);
        const startTop = ref(0);

        const isOverlap = (element1, element2) => {
            const rect1 = element1.value.getBoundingClientRect();
            const rect2 = element2.value.getBoundingClientRect();
            return !(
                rect1.right < rect2.left ||
                rect1.left > rect2.right ||
                rect1.bottom < rect2.top ||
                rect1.top > rect2.bottom
            );
        };

        const clearInput = () => {
            name.value = '';
            title.value = '';
            email.value = '';
            phone.value = '';
            phonecell.value = '';
            address.value = '';
            region.value = '';
            city.value = '';
            street.value = '';
            housenumber.value = '';
            zipcode.value = '';
        };

        const handlePaste = (event) => {
            const items = (event.clipboardData || event.originalEvent.clipboardData).items;
            for (const item of items) {
                if (item.type.indexOf('image') !== -1) {
                    const blob = item.getAsFile();
                    const reader = new FileReader();
                    reader.onload = (event) => {
                        imageSrc.value = event.target.result;
                    };
                    reader.readAsDataURL(blob);
                }
            }
        };

        const startDrag = (event) => {
            isDragging.value = true;
            startX.value = event.clientX || event.touches[0].clientX;
            startY.value = event.clientY || event.touches[0].clientY;
            startLeft.value = parseFloat(imageStyle.value.left);
            startTop.value = parseFloat(imageStyle.value.top);
            document.addEventListener('mousemove', onDrag);
            document.addEventListener('mouseup', stopDrag);
            document.addEventListener('touchmove', onDrag);
            document.addEventListener('touchend', stopDrag);
        };

        const onDrag = (event) => {
            if (isDragging.value) {
                const clientX = event.clientX || event.touches[0].clientX;
                const clientY = event.clientY || event.touches[0].clientY;
                const deltaX = clientX - startX.value;
                const deltaY = clientY - startY.value;
                imageStyle.value.left = `${startLeft.value + deltaX}px`;
                imageStyle.value.top = `${startTop.value + deltaY}px`;
            }
        };

        const stopDrag = () => {
            isDragging.value = false;
            document.removeEventListener('mousemove', onDrag);
            document.removeEventListener('mouseup', stopDrag);
            document.removeEventListener('touchmove', onDrag);
            document.removeEventListener('touchend', stopDrag);
        };

        onMounted(() => {
            window.addEventListener('paste', handlePaste);
            window.addEventListener('resize', adjustFontSize);
            generateQRCode();
        });

        onUnmounted(() => {
            window.removeEventListener('paste', handlePaste);
            window.removeEventListener('resize', adjustFontSize);
        });

        const adjustFontSize = () => {
            const ptKoeff = 0.74999943307122;
            let fontSize = parseFloat(textemail.value.style.fontSize) * ptKoeff;
            let textemailrect = textemail.value.getBoundingClientRect();
            const barrect = bar.value.getBoundingClientRect();

            if (isOverlap(textemail, bar)) {
                while (textemailrect.x + textemailrect.width + 1 > barrect.x) {
                    fontSize -= 0.2;
                    textemail.value.style.fontSize = fontSize + "pt";
                    textemailrect = textemail.value.getBoundingClientRect();
                }
            } else {
                do {
                    fontSize += 0.1;
                    textemail.value.style.fontSize = Math.min(fontSize, 8) + "pt";
                    textemailrect = textemail.value.getBoundingClientRect();
                    if (textemailrect.x + textemailrect.width + 2 >= barrect.x) {
                        fontSize -= 0.1;
                        textemail.value.style.fontSize = fontSize + "pt";
                        break;
                    }
                } while (textemailrect.x + textemailrect.width - 2 <= barrect.x && fontSize <= 8)
            }
        };

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

        const createPDF = async () => {
            const img1 = await html2canvas(cardbackside.value, { scale: 10 });
            const img2 = await html2canvas(cardfrontside.value, { scale: 10 });
            const doc = new jsPDF({
                unit: 'mm',
                userUnit: 300,
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
            imageSrc,
            imageStyle,
            startDrag,
            createPDF,
            regionoptions: [
                { text: 'г. Москва', value: 'г. Москва' },
                { text: 'Московская область', value: 'Московская область' },
                { text: 'Ленинградская область', value: 'Ленинградская область' },
                { text: 'Смоленская область', value: 'Смоленская область' },
                { text: 'Тверская область', value: 'Тверская область' },
                { text: 'Брянская область', value: 'Москваая область' },
                { text: 'Могилевская область', value: 'Псковская область' },
            ]
        }
    },

    directives: {
        clearonesc: {
            mounted(el) {
                el.addEventListener('keydown', async function (event) {
                    if (event.key === 'Escape') {
                        el.value = "";
                    }
                });
            }
        }
    }
}
</script>

<style>
/* Ваши стили остаются без изменений */
</style>