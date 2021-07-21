<template>
  <div id="viewerContent"></div>
</template>

<script>
  import { Stimulsoft } from 'stimulsoft-reports-js/Scripts/stimulsoft.designer'

  export default {
    name: 'HelloWorld',
    data() {
      return {

      }
    },
    mounted() {
      const testJson = [
        {
            "name": "T_Ashdod_000110_fb:9a:1f:1c:7b:61",
            "power": 254,
            "rssi": -93,
            "sys___active": 1,
            "sys___connected": 0
        },
        {
            "name": "T_Ashdod_000080_81:aa:6f:03:42:3f",
            "power": 254,
            "rssi": -87,
            "sys___active": 1,
            "sys___connected": 0
        }
    ]

        // var filterBy = 'sys___active';
        // var expression = '==';
        // var value = 0;

        Stimulsoft.Base.Localization.StiLocalization.setLocalization("../localization/he.xml");

        var report = Stimulsoft.Report.StiReport.createNewReport();
        report.loadFile("statusReport2.mrt");
        // var cultureName = "he-IL";
        // report.localizeReport(cultureName);

        var dataSet = new Stimulsoft.System.Data.DataSet("results");
        // dataSet.readJsonFile("report-data.json");
        dataSet.readJson(JSON.stringify(testJson));
        report.dictionary.databases.clear();
        report.regData("results", "results", dataSet);
        report.dictionary.synchronize();

        // report.renderAsync();

        var options = new Stimulsoft.Viewer.StiViewerOptions();
        options.appearance.rightToLeft = true;

        // console.log(report.getComponents());
        // var databand = report.renderedPages.getComponentByName("Tableroot");
        // databand.Filters.Add(new Stimulsoft.StiFilter(`root.${filterBy} ${expression} ${value}`));
        var viewer = new Stimulsoft.Viewer.StiViewer(options, "StiViewer", false);
        viewer.report = report;
        viewer.renderHtml("viewerContent");
    }
}
</script>

<style src="../../node_modules/stimulsoft-reports-js/Css/stimulsoft.viewer.office2010.blue.css"></style>
