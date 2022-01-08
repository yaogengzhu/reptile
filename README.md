# 利用工具获取一些网站图片
注：仅供学习参考, demo 参考网站有503 IP 限制了

## 技术工具
[-] playwright

[-] axios


```demo
const fs = require('fs');
const path = require('path');
const cheerio = require('cheerio');
const axios = require('axios');
const { chromium } = require('playwright');

/**
 * 初始化浏览器
 * @returns
 */
async function initChorme() {
    const browser = await chromium.launch({
        headless: false, // 是否开启无头浏览器
        slowMo: 300,
    }); // Or 'firefox' or 'webkit'.
    const page = await browser.newPage();
    return page;
}

/**
 * 获取页面dom, 初始化动作
 */
async function getHtmlContent(url) {
    const page = await initChorme();
    await page.goto(url, {
        waitUntil: 'load',
    });
    await page.waitForNavigation({
        waitUntil: 'load', // 页面加载完成
        timeout: 10000,
    });
    console.log('====================================');
    console.log('页面加载完成');
    console.log('====================================');
    const content = await page.content();
    return content;
}

/**
 *  初始化dom
 */
async function initDom(url, pageNumber) {
    const content = await getHtmlContent(url);
    const $ = await cheerio.load(content, { decodeEntities: false });
    $('.clearfix > li > a > img ').each(async (i, element) => {
        const imgUrl = $(element).attr('src');
        const title = $(element).attr('alt');
        await sleep(i * 1000); // 延时执行
        downloadPageImg(imgUrl, title.replace(/\s+/g, ''), pageNumber);
    });
}

function sleep(timer = 1000) {
    return new Promise((relove) => {
        setTimeout(() => {
            relove();
        }, timer);
    });
}
async function downloadPageImg(imgUrl, title, pageNumber) {
    const BASER_URL = 'https://pic.netbian.com' + imgUrl;
    const extendName = path.extname(BASER_URL);
    const imgPath =
        path.resolve(__dirname, `./第${pageNumber}页`) +
        `/${title}${extendName}`;
    // 写入流
    const ws = fs.createWriteStream(imgPath);
    axios.get(BASER_URL, { responseType: 'stream' }).then((res) => {
        // 管道
        console.log(imgPath + '写入完成');
        res.data.pipe(ws);
    });
    ws.on('finish', function () {
        ws.end();
    });
}

async function openAndDownload() {
    for (let i = 2; i < 3; i++) {
        const lcUrl = `https://pic.netbian.com/4kmeinv/index_${i}.html`;
        await sleep(i * 1000).then(() => {
            initDom(lcUrl, i);
        });
    }
}
openAndDownload();

```
