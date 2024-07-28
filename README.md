中文 | [English](README_en-US.md)

![introduce](https://socialify.git.ci/lctoolsweb/CialloTools/image?description=1&font=Source%20Code%20Pro&forks=1&issues=1&language=1&logo=https%3A%2F%2Fooo.0x0.ooo%2F2024%2F07%2F18%2FOR1UEB.png&name=1&owner=1&pulls=1&stargazers=1&theme=Light)

**这是[LunarCoreTools](https://github.com/lctoolsweb/LunarCoreTools/)适用与DanhengServer的分支，功能正在积极开发中，若你想要参与开发，请提交pr**  
# 免责声明

本项目只是用于学习TypeScript与vue相关的知识，切勿用于非法用途。请不要在无关的地方宣传LunarCoreTools。  
本项目完全遵守相关的开源协议，您可以根据自己的意愿自由地更改源码。
因用于商业用途而导致的任何法律后果，包括但不限于损失、责任、索赔或其他相关事宜，本项目的开发者概不负责。  
使用者需自行承担使用本项目的风险，并在遵循 MIT 许可证的前提下自行评估和处理相关的法律责任。  
**因不符合规定的商业行为，我们已经暂停示例页面的访问，使用者需自行搭建并承担可能带来的相关法律责任。**


## 耻辱柱
### Lunarcore及其他任何衍生工具都是免费软件。
+ 中国商人1:Q群377591124 Q号2914592234
+ 中国商人李贞璟 Q号57366054 yile一乐圈服，四处倒卖圈钱


# web版
## 多语言
页面会根据当前浏览器语言自动调整语言，目前仅支持中文，英文，且部分(bushi)翻译工作尚未完成，若您想为项目的国际化做出贡献，请提交pr。
The page will automatically adjust the language according to the current browser language, currently only supports Chinese, English, and some translation work has not yet been completed, if you want to contribute to the internationalization of the project, please submit a PR.

## 运行环境
*node 18*，其他版本不保证能稳定运行.

## 安装依赖(手动安装)
`
npm install
`

## 开发(在dev模式下响应较慢)
`
npm run dev
`

## 打包build运行(推荐)
`
npm run build
`
`
npm install http-server  
`
`
cd dist  
`
`
http-server
`
# 远程命令
请在`.env`文件中填入Dispatch服务地址与Adminkey，例如
```
VITE_DANHENG_DISPATCH_SERVER=http://127.0.0.1:443
VITE_DANHENG_ADMIN_KEY=KFC-Crazy-Thursday-vme50
```
Adminkey可以在服务器配置文件(config.json)中得到
# Ciallo～(∠・ω )⌒☆
点击特效与音效可以在` .env`文件中开启/关闭
```
VITE_CLICK_AUDIO=true
VITE_CLICK_TEXT=true
```
