---
published: true
title: "DoddleReport - Export type by parameter"
layout: post
---

[DoddleReport](http://doddlereport.codeplex.com/) is a fantastic library built by [Matt Hidinger](https://twitter.com/matthidinger) which is a very simple, install-and-go package to turn your datasets into exportable objects like CSV, PDF or Excel. It hasnt been updated for years, but it's still solid as a rock.

In using it through, there comes some limitations on exporting in MVC, as the built in routing style of {Action}.{Extension} causes issues within the app as the "." doesnt route properly.

To overcome this, I created an export that taps into the underlying types by using an "extension" parameter instead.

Just pass in xlsx, pdf etc, and it will call the underlying writer instead of using the built in doddle routing which was {action}.{extension}

<strong>Example Usage</strong>

<pre class="prettyprint">
http://localhost:80/Area/Controller/Action?extension={extension}
</pre>

In order to return this, just use the custom ReportResult (below) to return your results.

<strong>ReportResult ActionResult</strong>

<pre class="prettyprint">

    public class ReportResult : DoddleReport.Web.ReportResult
    {
        private readonly Report report;

        public ReportResult(Report report)
            : base(report)
        {
            this.report = report;
        }

        protected override string GetDownloadFileExtension(HttpRequestBase request, string defaultExtension)
        {
            var extension = request.Params["extension"];

            if (string.IsNullOrEmpty(extension))
                return defaultExtension;

            return "." + extension;
        }

        public override void ExecuteResult(ControllerContext context)
        {
            string defaultExtension = Config.Report.Writers.GetWriterConfigurationByFormat(Config.Report.DefaultWriter).FileExtension;

            var response = context.HttpContext.Response;

            var writerConfig = GetWriterFromExtension(context, defaultExtension);
            response.ContentType = writerConfig.ContentType;
            var writer = writerConfig.LoadWriter();

            if (!string.IsNullOrEmpty(FileName))
            {
                var extension = GetDownloadFileExtension(context.HttpContext.Request, defaultExtension);
                context.HttpContext.Response.AddHeader("content-disposition", string.Format("attachment; filename={0}{1}", FileName, extension));
            }

            writer.WriteReport(report, response.OutputStream);
        }

        private WriterElement GetWriterFromExtension(ControllerContext context, string defaultExtension)
        {
            string extension = GetDownloadFileExtension(context.RequestContext.HttpContext.Request, defaultExtension);

            var writerConfig = Config.Report.Writers.GetWriterConfigurationForFileExtension(extension);
            if (writerConfig == null)
                throw new InvalidOperationException(
                    string.Format(
                        "Unable to locate a report writer for the extension '{0}'. Did you add this fileExtension to the web.config for DoddleReport?",
                        extension));

            return writerConfig;
        }
    }
</pre>

Ballin'
