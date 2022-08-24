# excel导出

前端实现excel导出，无需在经过后端转一遍



具体代码如下：

```js
const ExcelJS = require('exceljs');

// 下载
const blobDownlod = (params, excelName) =>  {
  let url = window.URL.createObjectURL(params)
  let link = document.createElement('a')
  link.style.display = 'none'
  link.href = url
  link.setAttribute('download', excelName)
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link) // 下载完成移除元素
  window.URL.revokeObjectURL(url) // 释放掉blob对象
}

// excel格式设置
const exportExcelFile = (columNameArray, excelName, dataSource) => {
  // 创建工作簿
  const workbook = new ExcelJS.Workbook();
  // 添加sheet
  const worksheet = workbook.addWorksheet(excelName);
  // 设置 sheet 的默认行高
  worksheet.properties.defaultRowHeight = 20;
  // 设置列
  worksheet.columns = columNameArray
  
  // 添加行数据
  worksheet.addRows(dataSource);

  // 给表头添加背景色
  let headerRow = worksheet.getRow(1);
  headerRow.eachCell((cell) => {
    cell.fill = {
      type: 'pattern',
      pattern: 'solid',
      fgColor: {argb: 'dde0e7'},
    }
  })
  const fileType="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet,application/vnd.ms-excel"

  workbook.xlsx.writeBuffer().then(data=>{
    let blob = new Blob([data],{type:fileType})
    blobDownlod(blob, excelName)
  })
}
```

