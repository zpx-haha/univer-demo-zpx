<template>
  <div id="app">
    <div>
      <button @click="getData" title="print current workbook data to console">Get Data</button>
      <button @click="getAuthority" title="print current workbook data to console">权限</button>
      <button @click="getSelect" title="print current workbook data to console">下拉</button>
      <button @click="getSheet" title="print current workbook data to console">创建sheet</button>
    </div>
    <UniverSheet id="sheet" ref="univerRef" :data="data" @getWorkbook="getWorkbook" @getUniver="getUniver"/>
  </div>
</template>

<script>
import UniverSheet from './components/UniverSheet.vue'
import { ICommandService, univerInstanceService, IUniverInstanceService, IPermissionService} from "@univerjs/core";
import { getSheetCommandTarget, SheetsSelectionsService, AddWorksheetProtectionMutation, AddRangeProtectionMutation ,RangeProtectionPermissionEditPoint } from '@univerjs/sheets';
import { FUniver } from "@univerjs/facade";
import '@univerjs/sheets-data-validation/lib/index.css';
 
import { UniverDataValidationPlugin } from '@univerjs/data-validation';
import { UniverSheetsDataValidationPlugin, AddSheetDataValidationCommand  } from '@univerjs/sheets-data-validation';
import { DEFAULT_WORKBOOK_DATA } from './assets/default-workbook-data'

export default {
  name: 'App',
  components: {
    UniverSheet
  },
  data () {
    return {
      data: DEFAULT_WORKBOOK_DATA,
      workBook: null,
      univer: null,
      dropdownValidationRule:{
        type: 'list',
        showMenu: 'onFocus',
        inCellDropdown: true,
        allowBlank: false, // 禁止空白选择
        src: ['选项1', '选项2', '选项3'], // 下拉框的选项
        ranges: [{
          startRow: 1, // 起始行
          endRow: 2,   // 结束行
          startColumn: 0, // 起始列
          endColumn: 0,   // 结束列
        }],
      }
    }
  },
  created () {
    this.$nextTick(()=>{
      
      // this.sheetHandler()
    })
  },
  methods: {
    getUniver(data){
      this.univer = data
    },
    getWorkbook(data){
      this.workBook = data
    },
    getSheet(){
      const univerRef = this.$refs.univerRef.univer
      const univerAPI = FUniver.newAPI(univerRef);
      const activeWorkbook = univerAPI.getActiveWorkbook()
      console.log(activeWorkbook,'activeWorkbook')
      activeWorkbook.create('asd')
      // const newSheet = this.workBook.createSheet('hahaha')
      // console.log(this.workBook,'this.workBook')
      // console.log(univerAPI.getUniverSheet('sheet-01'))
      // univerAPI.createUniverSheet({
      //   id:"123ads",
      //   name:'maysheet',
      // })
    },
    getSelect(){
      const univerRef = this.$refs.univerRef.univer
      const univerAPI = FUniver.newAPI(univerRef);
      // const sheet = univerAPI.getActiveWorkbook().getActiveSheet();
      // console.log(sheet,'sheetsheet')
      // // get range
      // const range = sheet.getRange(0, 0, 1, 1);
      
      // // build data validation
      // const dataValidationBuilder = FUniver.newDataValidation();
      // const dataValidation = dataValidationBuilder.requireCheckbox().build();
      // console.log(dataValidationBuilder,'dataValidationBuilder')
      // console.log(range,'rangerangerangerange')
      // set data validation
      // range.setDataValidation(dataValidation);
      // end
      // const accessor = univerRef.__getInjector();
      // const commandService = accessor.get(ICommandService);
      // const univerInstanceService = accessor.get(IUniverInstanceService);
      // const target = getSheetCommandTarget(univerInstanceService);
      // const sheetSelectionManagerService = accessor.get(SheetsSelectionsService);
      // if (!target) {
      //   return;
      // }
      // const { unitId, subUnitId } = target;
      // const ranges = sheetSelectionManagerService.getCurrentSelections().map(selection => selection.range);
      // commandService.executeCommand(AddSheetDataValidationCommand.id, {
      //   unitId,
      //   subUnitId,
      //   rule: this.dropdownValidationRule,
      // });
      // end

      const sheet = univerAPI.getActiveWorkbook().getActiveSheet();
      const dataValidationBuilder = FUniver.newDataValidation();
      const dropdownOptions = ['选项1', '选项2', '选项3'];
      const defaultValue = '选项1';
      const dataValidation = dataValidationBuilder.requireValueInList(dropdownOptions).build();

      // 获取单元格范围
      const range = sheet.getRange(3, 5, 1, 1);
      // 设置数据验证
      range.setDataValidation(dataValidation);
      range.setValue(defaultValue);
    },

    getAuthority(){
      const univerRef = this.$refs.univerRef.univer
      // const univerAPI = FUniver.newAPI(univerRef);
      // 禁用整个表
      // univerAPI.getActiveWorkbook().setEditable(false)
      const accessor = univerRef.__getInjector();
      
      const permissionService = accessor.get(IPermissionService);
      const commandService = accessor.get(ICommandService);
      const sheetSelectionManagerService = accessor.get(SheetsSelectionsService);
      const univerInstanceService = accessor.get(IUniverInstanceService);
      const target = getSheetCommandTarget(univerInstanceService);
      if (!target) {
        return;
      }
      const { unitId, subUnitId } = target;
      const ranges = sheetSelectionManagerService.getCurrentSelections().map(selection => selection.range);
      // console.log(AddRangeProtectionMutation.id,'AddWorksheetProtectionMutation.id')
      // console.log(ranges,'ranges')
      commandService.executeCommand(AddRangeProtectionMutation.id, {
        unitId,
        subUnitId,
        rules: [{
          permissionId: "3xtfxG1",
          name: "sheet1",
          unitType: 3,
          unitId,
          subUnitId,
          ranges,
          id: 'rule1'
        }],
      });
      permissionService.updatePermissionPoint(new RangeProtectionPermissionEditPoint(unitId, subUnitId, "3xtfxG1").id, false)
    },
    getData () {
      const univerRef = this.$refs.univerRef.univer
      const univerAPI = FUniver.newAPI(univerRef);
      const savedData = univerAPI.getActiveWorkbook().save();
      console.log(JSON.stringify(savedData),'savedDatasavedData')
      console.log(this.workBook.save())
      // const result = this.$refs.univerRef?.getData();
      // console.log(JSON.stringify(result));
      // console.log(JSON.stringify(result, null, 2));
      // const univerAPI = FUniver.newAPI(this.$refs.univerRef.univer);
      // console.log(univerAPI,'univerAPIuniverAPI')
      // univerAPI.onCommandExecuted((command)=>{
      //   const { id, type, params } = command;
      //   console.log(id, type, params,'id, type, params')
      //   // 在命令执行前执行自定义逻辑
      // })
    },
    sheetHandler(){
      const univerAPI = FUniver.newAPI(this.$refs.univerRef.univer);
      // univerAPI.onCommandExecuted((command) => {
      //   console.log(command,'commandcommand')
      //   if(command.id === 'sheet.operation.set-cell-edit-visible' && command.params.visible){
      //     console.log('Cell edit visible')
      //   }
      // })
      // univerAPI.onCommandExecuted((command)=>{
      //   console.log(command,'commandcommand')
      // })
    }
  }
}
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
}

#app {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

#sheet {
  /** The height of the Union uses the height of the parent container */
  flex: 1;
}
</style>
