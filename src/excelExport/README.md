### 关于

> 前端生成excel而不是csv最主要目的都是为了解决csv不能实现单元格合并的问题，要不然直接导出csv文件就好了，何必引入几百kb的插件。本文导出excel文件引用了[js-xlsx插件](https://github.com/SheetJS/js-xlsx)

1. 导出csv文件(csv文件无法合并单元格)
```
/**
 * @param value //[{'姓名':'小明','成绩','100'}]
 * @param columns //['姓名','成绩']
 * @param exportFilename // '成绩统计表'
 */
var csvSeparator = ','; 
function exportCSV(value, columns, exportFilename) {
    const data = value;
    let csv = '\ufeff';
    // headers
    for (let i = 0; i < columns.length; i++) {
    const column = columns[i];
    csv += '"' + (column.header || column) + '"';
    if (i < (columns.length - 1)) {
        csv += csvSeparator;
    }
    }
    // body
    data.forEach((record) => {
    csv += '\n';
    for (let i_1 = 0; i_1 < columns.length; i_1++) {
        const column = columns[i_1];
        csv += '"' + resolveFieldData(record, column) + '"';
        if (i_1 < (columns.length - 1)) {
        csv += csvSeparator;
        }
    }
    });
    console.log(csv);
    // return;
    const blob = new Blob([csv], {
    type: 'text/csv;charset=utf-8;'
    });
    if (window.navigator.msSaveOrOpenBlob) {
    navigator.msSaveOrOpenBlob(blob, exportFilename + '.csv');
    } else {
    const link = document.createElement('a');
    link.style.display = 'none';
    document.body.appendChild(link);
    if (link.download !== undefined) {
        link.setAttribute('href', URL.createObjectURL(blob));
        link.setAttribute('download', exportFilename + '.csv');
        link.click();
    } else {
        csv = 'data:text/csv;charset=utf-8,' + csv;
        window.open(encodeURI(csv));
    }
    document.body.removeChild(link);
    }
}

function resolveFieldData(data, field) {
    if (data && field) {
    if (field.indexOf('.') === -1) {
        return data[field];
    } else {
        const fields = field.split('.');
        let value = data;
        for (let i = 0, len = fields.length; i < len; ++i) {
        if (value === null) {
            return null;
        }
        value = value[fields[i]];
        }
        return value;
    }
    } else {
    return null;
    }
}

function export2CSV() {
    var value = [{'姓名':'小明','成绩':'100'},{'姓名':'小明2','成绩':'1003'}];
    var columns = ['姓名','成绩'];
    var exportFilename = '成绩统计表';
    exportCSV(value, columns, exportFilename);
}
```


2. 导出excel文件，带合并单元格的(需先引入js-xlsx插件)
> 需要注意的地方就是被合并的单元格要用null预留出位置，否则后面的内容（本例中是第四列其它信息）会被覆盖。
```
function export2Excel() {
    var aoa = [
		['主要信息', null, null, '其它信息'], // 特别注意合并的地方后面预留2个null
		['姓名', '性别', '年龄', '注册时间'],
		['张三', '男', 18, '2019-03-04'],
        ['李四', '女', 22, null],
		['李四', '男', 22, null]            
	];
	var sheet = XLSX.utils.aoa_to_sheet(aoa);
	sheet['!merges'] = [
		// 设置A1-C1的单元格合并 设置D3-D5合并
        {s: {r: 0, c: 0}, e: {r: 0, c: 2}},
        {s: {r: 2, c: 3}, e: {r: 4, c: 3}},
	];
	openDownloadDialog(sheet2blob(sheet), '单元格合并示例.xlsx');
}


// 将一个sheet转成最终的excel文件的blob对象，然后利用URL.createObjectURL下载
function sheet2blob(sheet, sheetName) {
	sheetName = sheetName || 'sheet1';
	var workbook = {
		SheetNames: [sheetName],
		Sheets: {}
	};
	workbook.Sheets[sheetName] = sheet;
	// 生成excel的配置项
	var wopts = {
		bookType: 'xlsx', // 要生成的文件类型
		bookSST: false, // 是否生成Shared String Table，官方解释是，如果开启生成速度会下降，但在低版本IOS设备上有更好的兼容性
		type: 'binary'
	};
	var wbout = XLSX.write(workbook, wopts);
	var blob = new Blob([s2ab(wbout)], {type:"application/octet-stream"});
	// 字符串转ArrayBuffer
	return blob;
}

function s2ab(s) {
    var buf = new ArrayBuffer(s.length);
    var view = new Uint8Array(buf);
    for (var i=0; i!=s.length; ++i) view[i] = s.charCodeAt(i) & 0xFF;
    return buf;
}


/**
 * 通用的打开下载对话框方法，没有测试过具体兼容性
 * @param url 下载地址，也可以是一个blob对象，必选
 * @param saveName 保存文件名，可选
 */
function openDownloadDialog(url, saveName)
{
    if(typeof url == 'object' && url instanceof Blob)
    {
        url = URL.createObjectURL(url); // 创建blob地址
    }
    var aLink = document.createElement('a');
    aLink.href = url;
    aLink.download = saveName || ''; // HTML5新增的属性，指定保存文件名，可以不要后缀，注意，file:///模式下不会生效
    var event;
    if(window.MouseEvent) event = new MouseEvent('click');
    else
    {
        event = document.createEvent('MouseEvents');
        event.initMouseEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
    }
    aLink.dispatchEvent(event);
}

```

感谢

参考文章: https://www.cnblogs.com/liuxianan/p/js-excel.html