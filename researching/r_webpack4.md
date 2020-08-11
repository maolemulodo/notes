Để nghiên cứu rõ về Webpack thì cách hiệu quả nhất có lẽ phải tìm đọc tài liệu ở trang chính thức của Webpack. Nên đây chỉ là bài giới thiệu tổng quan nhất về Webpack (Phiên bản Webpack được giới thiệu ở đây là Webpack4)

## Webpack là gì?

Cách hiểu ngắn gọn thì Webpack gom các files (CSS, JS, SASS, JPG, SVG, PNG, ...) và những liên quan của chúng thành các nhóm file.

![webpack understand](../assets/images/webpack.png?raw=true)

## Cài đặt Webpack

Sử dụng `npm` để tạo một `package.json` file trong thư mục project

```bash
npm init
```

Tiếp theo ta cài đặt webpack với câu lệnh sau

```bash
npm install --save-dev webpack webpack-dev-server webpack-cli
```

## Cấu hình Webpack

Tạo file `webpack.config.js` trong project's folder

### Entry

Chúng ta đơn giản thông báo với webpack rằng hãy để mắt xem file index.js

```javascript
module.exports = {
  entry: "./src/index.js",
};
```

Chúng ta có thể có nhiều hơn một entry point, điều này sẽ được thảo luận sau

### Output

Khai báo đường dẫn output bundle file cho webpack. `/dist/main.js, là đường dẫn mặc định`. Chúng ta có thể thay đổi filename bên trong output, path là nơi file sẽ được output.

```javascript
const path = require("path");
module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.js",
  },
};
```

Chỉnh sửa command tại phần "script" trong file "package.json"

```json
"script" : {
   "start" : "webpack --config webpack.config.js"
}
```

### Mode

Với mode là `development` file output sẽ không minify điều này giúp chúng ta dễ đọc code của file đã được output

```javascript
const path = require("path");
module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.js",
  },
};
```

## Loader

Webpack không chỉ đóng gói các file javascript, mà nó có thể đóng gói các loại file khác nữa, như chúng ta đã nói về tính năng webpack ở phía trên. Và chía khóa là của việc đóng gói các file khác chính là `Loader` - Tham khảo loader tại [đây](https://webpack.js.org/loaders/)

### CSS loader

Chúng ta có thể định nghĩa loader là một mảng của `rules` và định nghĩa này nằm trong đối tượng `module`. Để áp dụng loader chúng ta bắt đầu thử tạo một file `main.css` và cài đặt gói `style-loader` và `css-loader` cho ứng dụng của chúng ta

```bash
npm install --save-dev style-loader css-loader
```

Tiếp theo chúng ta sẽ update file webpack.config.js của mình sau. (thêm module vào file cấu hình)

```javascript
const path = require("path");
module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.js",
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
};
```

Quá trình đọc của webpack là từ dưới lên trên, nó bắt đầu thực thi `css-loader` trước, sau đó đến `style-loader`, ở đây `css-loader` sẽ chuyển đổi file các file css đến mã Javascript và `style-loader` đưa style vào DOM

Đối với **sass**, chúng ta phải sự dụng loader khác là `sass-loader`, ngoài ra sass-loader cũng yêu cầu `node-sass`

```bash
npm install sass-loader node-sass webpack --sav-dev
```

Chúng ta lại tiếp tục update webpack config file

```javascript
const path = require("path");
module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.js",
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          "style-loader", // Injects style into DOM
          "css-loader", // Turns CSS into JS
          "sass-loader", // Turns SCSS into CSS
        ],
      },
    ],
  },
};
```

Đừng quên thêm file main.css và main.scss vào file index.html và style được định nghĩa trong main. css/main.scss sẽ được thêm vào trong Html DOM. Bây giờ chúng ta cần tạo trong một CSS độc lập.

## Cache Busting and Plugins

Thông thường trình duyệt sẽ tự cache lại tất cả mã nguồn của chúng ta, và khi file có sự thay đổi thì user không thể download lại file đó. Trong trường hợp này ta muốn file `main.js` sẽ được tải xuống khi có sự thay đổi. Vì thế chúng ta sẽ dùng một phương thức hashing đặc biệt để thay đổi tên của file `main.js` thành `main.abc123.js`

Với phương thức này khi không có bất cứ sự thay đổi nào trên file `main.js` thì tên file sẽ được giữ nguyên là `main.abc123.js`. Nhưng khi có sự thay đổi thì nó sẽ cung cấp cho ta một file main với tên đã được cập nhật `main.vnas28r9ysd.js` và vì thế browser có thể download nó về.

Điều này rất hữu ích khi chúng ta thêm vendor vào source code của chúng ta. Chúng ta không nhất thiết phải download mỗi lần như thế, bởi vì thực thế chúng không thay đổi thường xuyên các files vendor và nó có thể được cached bởi browser.

```javascript
module.exports = {
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.[contentHash].js",
  },
};
```

Vấn đề của chúng ta là làm sao chúng ta có thể bỏ file main mới được sinh ra này vào trong `index.html` vì chúng ta sẽ không biết được chính xác tên file sẽ được sinh ra là gì. Webpack sẽ làm nó như sau, webpack tạo ra đoạn script cuối file HTML, đoạn script này tự động được thêm vào bằng cách sử dụng plugin.

## Plugins

Plugins được sử dụng để tùy chỉnh cấu hình webpack với nhiều cách thức, bạn có thể tham khảo các plugins ở [đây](https://webpack.js.org/plugins/).

### Html webpack plugin

Plugin Html này đơn giản chỉ tạo ra file HTML để phục vụ đóng gói webpack của bạn. Plugin này thực sự hữu ích cho trường hợp thêm `contentHash` của chúng ta.

```bash
npm install --save-dev html-webpack-plugin
```

Update webpack.config.js

```javascript
const path = require("path");
module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.[contentHash].js",
  },
  plugins: [new HtmlWebpackPlugin()],
};
```

Webpack lúc này sẽ tạo ra file `index.html` và đưa nó vào trong thự mục `dist` cho chúng ta, và đồng thời thêm đoạn mã scrips có chưa file main đã được tạo với tên đã được hash.

```html
<!DOCTYPE html>
<html>
<head>
   <meta charset=”UTF-8">
   <title>Webpack App</title>
</head>
<body>
   <script type=”text/javascript” src=”main.170skb99wrjcb.js”></script>
</body>
</html>
```

Nhưng file `index.html` không chứa mã nguồn file `index.html` của chúng ta. Vậy nên chúng ta cần cung cấp template cho webpack config

```javascript
const path = require("path");
module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.[contentHash].js",
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
    }),
  ],
};
```

Phần trước chúng ta đã dùng css-loader để thêm css file vào DOM. Thực tế, chúng ta cần css file độc lập, để làm điều này chúng ta sử dụng `mini-css-extract-plugin`

```bash
npm install mini-css-extract-plugin
```

Plugin này sẽ trích xuất các css file thành một file riêng biệt, khi này webpack config sẽ dược update lại như sau

```javascript
const path = require("path");
module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.[contentHash].js",
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          "style-loader", // Injects style into DOM
          "css-loader", // Turns CSS into JS
          "sass-loader", // Turns SCSS into CSS
        ],
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
    }),
    new MiniCssExtractPlugin(),
  ],
};
```

Và chúng ta cũng đã thêm file css mới vào HTML plugin. Như vậy mỗi lần chúng ta có file css mới được tạo ra chúng ta sẽ không cần thêm nó vào file index một cách thủ công nữa, nó sẽ được HTLM webpack plugin thêm vào tự động.

## Splitting Code Dev and Production Mode

Chế độ dev và production đã được giới thiệu trong webpack4. Chúng ta sẽ tách cấu hình webpack thành webpack config cho dev và webpack config cho production.

Từn nãy đến giờ chúng ta có 1 file webpack config chứa rất nhiều thông tin chung cho cả dev và production. Vậy nên, tiếp theo chúng ta sẽ chia nó thành 3 files.

- Webpack config cho dev
- Webpack config cho production
- Webpack config common

Chúng ta sẽ không build folder dist tại môi trường dev - `webpack-dev-server` sẽ giúp chúng ta load những file này từ bộ nhớ. Chúng ta chỉ gói gọn file cuối cùng đến môi trường production.

Chúng ta sẽ tạo các file như sau

- webpack.dev.js
- webpack.prod.js
- webpack.common.js

Tên config có thể tùy bạn lựa chọn - bạn có thể sử dụng bất cứ tên nào bạn muốn.

Tiếp theo cài đặt gói `webpack-dev-server`

```bash
npm install --save-dev webpack-dev-server
```

Gói `webpack-dev-server` hỗ trợ cho chúng ta live reloadings (tức browser sẽ tự động reload lại page khi có bất cứ sự thay đổi nào từ source code của chúng ta). Nó cũng cung cấp khả năng truy cập nhanh đến bộ nhớ để truy cập vào nội dung config của webpack. Cái này chúng ta sử dụng cho môi trường dev

Tiếp theo chúng ta cái đặt gói `webpack-merge`

```bash
npm install --save-dev webpack-merge
```

Gói `webpack-merge` cung cấp merging functions đó là nối các arrays lại với nhau, tạo ra một đối tược objects mới với giá trị được merges từ nhiều object. Giá trị trả về được gán vào một function.

Bắt đầu chia tách file webpack config, chúng ta định hình sự khác nhau giữa các cấu hình như sau:

- Đối với common file chúng ta không cần thông tin output
- Chúng ta không đưa `contentHash` vào cấu hình dev

### webpack.common.js

```javascript
const path = require("path");
var HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/index.js",
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/template.html",
    }),
  ],
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          "style-loader", // Inject styles into DOM
          "css-loader", // Turns css into commonjs
          "sass-loader", // Turns sass into css
        ],
      },
    ],
  },
};
```

### webpack.dev.js

```javascript
const path = require("path");
const common = require("./webpack.common");
const merge = require("webpack-merge");

module.exports = merge(common, {
  mode: "development",
  output: {
    filename: "main.js",
    path: path.resolve(__dirname, "dist"),
  },
});
```

### webpack.prod.js

```javascript
const path = require("path");
const common = require("./webpack.common");
const merge = require("webpack-merge");

module.exports = merge(common, {
  mode: "production",
  output: {
    filename: "main.[contentHash].js",
    path: path.resolve(__dirname, "dist"),
  },
});
```

Tiếp theo chúng ta phải cấu hình cho phần `webpack-dev-server`. Chúng ta cũng cần thêm script trên `package.json` file để có thể thực thi cho môi trường dev hay production

```json
{
  "script": {
    "start": "webpack-dev-server --config webpack.dev.js --open",
    "build": "webpack --config webpack.prod.js"
  }
}
```

Sử dụng `npm start` trên môi trường dev, và `npm build` cho production, khi run cho môi trường production bạn sẽ thấy các files đã minifying trong build folder. khi run cho môi trường dev trong bộ nhớ thay đổi `--open`argument sẽ tự động mở trình duyệt khi mọi thứ đã sẵn sàng.

## Multiple entries

Oops!, bây giờ chúng ta có các vendor scrips như bootstrap và chúng ta muốn thêm nó vào entry của webpack. Đừng lo lắng, nó không khó để setting đâu. Chúng ta cần tách các bundles thành `main.js` và `vendor.js`. Theo đó thì entry point trong `webpack.common.js` sẻ được update lại như sau:

```javascript
module.exports = {
  entry: {
    main: "./src/index.js",
    vendor: "./src/vendor.js",
  },
};
```

Chúng ta cũng cấu hình lại output để tạo ra 2 bundle khác nhau cho config của dev và production

```javascript
// webpack.dev.js
module.exports = merge(common, {
  mode: "development",
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
});

// webpack.prod.js
module.exports = merge(common, {
  mode: "production",
  output: {
    filename: "[name].[contentHash].js",
    path: path.resolve(__dirname, "dist"),
  },
});
```

Với setting trên, webpack sẽ tạo ra 2 file bundle khác nhau trong cả hai môi trường dev và productions. Bạn có thể nhìn thấy các bundle là `main.bundle.js` và `vendor.bundle.js` trong chế độ development. Còn đối với chế độ productions bạn sẽ thấy 2 file là `main.88xxxdd.js` và `vendor.909090dddd.js`

## Conclusion

Bạn cũng có thể sử dụng images và các loại nội dung khác trong các dự án của bạn. Để làm điều này chúng ta cần phải thêm các gói packages theo yêu cầu, nếu không bạn sẽ được thông báo lỗi "You may need an appropriate loader to handle this file type" trên cấu hình webpack của bạn.

Refer to the original guide: https://medium.com/better-programming/webpack-4-the-complete-guide-af1b1e2e3f7a
