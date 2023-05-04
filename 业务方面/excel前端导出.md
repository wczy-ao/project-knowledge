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



# 合并单元格导出

```js
const ExcelJS = require('exceljs');

const transToFile = async(blob, fileName, fileType) => {
  return new window.File([blob], fileName, {type: fileType})
}
/**
 * 
 * @param {Blob} params Blob 对象
 * @param {String} excelName 导出名
 */
 export const blobDownlod = (params, excelName) =>  {
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

// 创建合并单元格记录
const mergeCellsMap = {}

// 获得间隔合并单元格数
const intervalsMergeCells = (data) => {
  const { pointNameVo = [] } = data
  return pointNameVo.length || 1
}


// 合并单元格
const mergeCells = (worksheet, data) => {
  // 合并表头结果列
  // worksheet.mergeCells( 1, 3,  1, 5);
  let lastRow = 1 // 上一个间隔结束行的数
  for(let i = 0; i < data.length; i++) {
    let intervalsMergeCellCount = intervalsMergeCells(data[i])
    if (i === 0) {
      worksheet.mergeCells( 2, 1,  1 + intervalsMergeCellCount, 1);
      mergeCellsMap[i] = {start: 2, end: 1 + intervalsMergeCellCount}
    } else {
      worksheet.mergeCells( lastRow + 1, 1,  lastRow + intervalsMergeCellCount, 1 );
      mergeCellsMap[i] = {start: lastRow + 1, end: lastRow + intervalsMergeCellCount }
    }
    lastRow = lastRow + intervalsMergeCellCount
  }
}
// a:{},b:null,c:null
const judgeABC = (opt) => {
  const {a,b,c} = opt
  if ( (a && a.value) || (b && b.value) || (c && c.value)) {
    return true
  } else {
    return false
  }
}
// 往单元格里设置值
const setChildrenRowValue = (data, index, worksheet, keyToValue) => {
  const arr = ['a', 'b', 'c']
  let startIndex = mergeCellsMap[index].start
  data.forEach((item, zIndex) => {
    const { pointName = '', value = null} = item
    let abcHasValue = judgeABC(item)
    if (value != null && value != undefined) {
      const {unit} = value
      if (value.value) {
        let gradleNoABC = value.gradle ? '(' + value.gradle + ')' : ''
        worksheet.getCell(`${keyToValue.value}${startIndex + zIndex}`).value = value.value + unit + gradleNoABC
      } else {
        worksheet.getCell(`${keyToValue.value}${startIndex + zIndex}`).value = '暂无'
      }
    } else if (abcHasValue && !value){
      arr.forEach(zItn => {
        if (item[zItn].value) {
          let {value,  unit} = item[zItn]
          let gradleABC = item[zItn].gradle ? '(' + item[zItn].gradle + ')' : ''
          worksheet.getCell(`${keyToValue[zItn]}${startIndex + zIndex}`).value =zItn.toLocaleUpperCase() + '：' + value + unit + gradleABC 
        } else {
          worksheet.getCell(`${keyToValue[zItn]}${startIndex + zIndex}`).value = '暂无'
        }
      })
    } else if (!abcHasValue) {
      worksheet.getCell(`${keyToValue.value}${startIndex + zIndex}`).value = '暂无'
    }
    // 设置点位名称值
    worksheet.getCell(`${keyToValue.pointName}${startIndex + zIndex}`).value = pointName
  })
}

const setRowValue = (worksheet, data) => {
  // 对应excel上面的列
  const keyToValue = {
    pointName: 'B',
    a: 'C',
    b: 'D',
    c: 'E',
    value: 'C'
  }
  for(let i = 0; i < data.length; i++) {
    worksheet.getCell(`A${mergeCellsMap[i].start}`).value = data[i].intervals
    worksheet.getCell(`A${mergeCellsMap[i].start}`).alignment = {vertical: "middle", horizontal:  "center"}

    // 设置子单元格值
    setChildrenRowValue(data[i].pointNameVo, i, worksheet, keyToValue)    
  }
}

/**
 * @param {Array} columNameArray 导出的列
 * @param {String} excelName 导出文件名
 * @param {JSON} dataSource 导出的数据
 */

export const excelExportOrDownload = (columNameArray, excelName, dataSource, type = 'download') => {
  // 创建工作簿
  const workbook = new ExcelJS.Workbook();
  // 添加sheet
  const worksheet = workbook.addWorksheet(excelName);
  // 设置 sheet 的默认行高
  worksheet.properties.defaultRowHeight = 20;
  // 表头居中
  columNameArray.forEach((item, index) => {
    worksheet.getCell(1, index+1, 1, index+1).alignment = {vertical: "left", horizontal:  "center"}
  })
  
  // 合并
  mergeCells(worksheet, dataSource )
  // 设置值
  setRowValue(worksheet, dataSource)
  // 设置列
  worksheet.columns = columNameArray
  worksheet.verticalCentered = true
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

  return new Promise((resolve)=> {
    workbook.xlsx.writeBuffer().then(async data=>{
      let blob = new Blob([data],{type:fileType})
      if (type === 'download') {
        blobDownlod(blob, excelName)
      } else if (type === 'upload') {
        let textContain = await transToFile(blob, "mapname.xls", "application/vnd.ms-excel")
          console.log({textContain});
          resolve(textContain)
      }
    })
  })
  
}
```

