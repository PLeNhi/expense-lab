# Specs: Add theme and mascot for website

## Style:

    - Website theo style hoạt hình dễ thương
    - Tone màu đỏ đô và nâu xám.

## Mascot

    - Dùng mascot được show ở trang home của website tên sikh từ thư viện https://koboyo.com/page-mascot
    - Cách làm như sau:
        1. Install the component: npm i page-mascot
        2. Download the two sheets: A character is two files. Put both in public/mascots. được lưu trong folder: /public/mascot
        3. Point the component at them
            import { Mascot } from 'page-mascot'

            <Mascot
            directions="/mascots/sikh-directions.webp"
            reactions="/mascots/sikh-reactions.webp"
            />
