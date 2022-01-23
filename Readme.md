<!-- default badges list -->
![](https://img.shields.io/endpoint?url=https://codecentral.devexpress.com/api/v1/VersionRange/128532327/13.1.4%2B)
[![](https://img.shields.io/badge/Open_in_DevExpress_Support_Center-FF7200?style=flat-square&logo=DevExpress&logoColor=white)](https://supportcenter.devexpress.com/ticket/details/E2355)
[![](https://img.shields.io/badge/📖_How_to_use_DevExpress_Examples-e9f6fc?style=flat-square)](https://docs.devexpress.com/GeneralInformation/403183)
<!-- default badges end -->
<!-- default file list -->
*Files to look at*:

* [Default.aspx](./CS/WebSite/Default.aspx) (VB: [Default.aspx](./VB/WebSite/Default.aspx))
* [Default.aspx.cs](./CS/WebSite/Default.aspx.cs) (VB: [Default.aspx.vb](./VB/WebSite/Default.aspx.vb))
<!-- default file list end -->
# ASP.NET Web Forms - ASPxComboBox in Cascading Combo Boxes
<!-- run online -->
**[[Run Online]](https://codecentral.devexpress.com/e2355/)**
<!-- run online end -->

This demo shows how to use the [ASPxComboBox](https://docs.devexpress.com/AspNet/DevExpress.Web.ASPxComboBox) editor type to implement two cascading combo boxes. In the demo, a selection in the first combo box (Country) filters the item list of the second combo box (City).

![example demo](demo.gif)

To implement cascading [ASPxComboBox](https://docs.devexpress.com/AspNet/DevExpress.Web.ASPxComboBox) editors, use the following technique:  
Respond to a change of the first combo box's selected value (in the client [SelectedIndexChanged](https://docs.devexpress.com/AspNet/js-ASPxClientComboBox.SelectedIndexChanged) event) and initiate a round trip to the server for the second combo box to filter its items (use a combination of the client [PerformCallback](https://docs.devexpress.com/AspNet/js-ASPxClientCallback.PerformCallback(parameter)) method and the server [Callback](https://docs.devexpress.com/AspNet/DevExpress.Web.ASPxCallback.Callback) event).


## 1. Setup combo boxes and their data sources
Create two [ASPxComboBox](https://docs.devexpress.com/AspNet/DevExpress.Web.ASPxComboBox) editors and define their data sources. In the second combo box's data source, implement a [parameter](https://docs.microsoft.com/en-us/dotnet/api/system.web.ui.webcontrols.parameter?view=netframework-4.8) to dynamically filter list items based on the first combo box's selected value.

```html
<dx:ASPxComboBox runat="server" ID="CountryCombo" ClientInstanceName="countryCombo" DataSourceID="CountryDataSource"...>
<dx:ASPxComboBox runat="server" ID="CityCombo" ClientInstanceName="cityCombo" DataSourceID="CityDataSource" OnCallback="CityCombo_Callback"...>

<asp:SqlDataSource ID="CountryDataSource" runat="server" ...
    SelectCommand="SELECT * FROM [Countries]"/>

<asp:SqlDataSource ID="CityDataSource" runat="server" ...
    SelectCommand="
        SELECT ct.City 
        FROM [Cities] ct, [Countries] cntr 
        WHERE (ct.CountryId = cntr.CountryId) AND (cntr.Country = @CountryName) 
            order by ct.City">
    <SelectParameters>
        <asp:Parameter Name="CountryName" />
    </SelectParameters>/>
```
## 2. On client: Respond to a selection change and initiate a callback 
Handle the first combo box's client-side [SelectedIndexChanged](https://docs.devexpress.com/AspNet/js-ASPxClientComboBox.SelectedIndexChanged) event. In the event handler, call the client-side [PerformCallback](https://docs.devexpress.com/AspNet/js-ASPxClientCallback.PerformCallback(parameter)) method of the second combo box. This initiates a callback to the server for the second  editor to filter its item list. In the [PerformCallback](https://docs.devexpress.com/AspNet/js-ASPxClientCallback.PerformCallback(parameter)) method's [parameter](https://docs.devexpress.com/AspNet/js-ASPxClientCallback.PerformCallback(parameter)#parameters), pass the first combo box's selected value to use it as a filter criterion on the server.

``` html
<script>
    function OnCountryChanged(selectedValue) {
        cityCombo.PerformCallback(selectedValue);
    }
</script>
    ...
<dx:ASPxComboBox runat="server" ID="CountryCombo" ClientInstanceName="countryCombo" ...>
    <ClientSideEvents SelectedIndexChanged="function(s,e){OnCountryChanged(s.GetSelectedItem().value.toString());}"/>
    ...
```

## 3. On server: Filter list items
Handle the second combo box's server-side [Callback](https://docs.devexpress.com/AspNet/DevExpress.Web.ASPxCallback.Callback) event that is generated in response to a call to the client-side [PerformCallback](https://docs.devexpress.com/AspNet/js-ASPxClientCallback.PerformCallback(parameter)) method. In the handler, use the event argument's [Parameter](https://docs.devexpress.com/AspNet/DevExpress.Web.CallbackEventArgsBase.Parameter) property to obtain the first combo box's value passed from the client side. Use this value to filter the second combo box's data source.

```c#
protected void CityCombo_Callback(object source, DevExpress.Web.CallbackEventArgsBase e) {
    FillCityCombo(e.Parameter);
}
protected void FillCityCombo(string country) {
    if (string.IsNullOrEmpty(country)) return;

    //Update the data source bound to the second combo box.
    CityDataSource.SelectParameters["CountryName"].DefaultValue = country;
    CityCombo.DataBind();

    //Select the first city in a list.
    CityCombo.SelectedIndex = 0;
}
```
