Example of creating new classes as extensions of Windows Server Computer and discovering instances of these classes using RegEx matches on the computer name.

The following is the full XML this MP project builds:

<?xml version="1.0" encoding="utf-8"?>
<ManagementPack SchemaVersion="2.0" ContentReadable="true" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Manifest>
    <Identity>
      <ID>Computer.Name.Example.MP</ID>
      <Version>1.0.0.44</Version>
    </Identity>
    <Name>Computer Name Example MP</Name>
    <References>
      <Reference Alias="Windows">
        <ID>Microsoft.Windows.Library</ID>
        <Version>7.5.8501.0</Version>
        <PublicKeyToken>31bf3856ad364e35</PublicKeyToken>
      </Reference>
      <Reference Alias="System">
        <ID>System.Library</ID>
        <Version>7.5.8501.0</Version>
        <PublicKeyToken>31bf3856ad364e35</PublicKeyToken>
      </Reference>
    </References>
  </Manifest>
  <TypeDefinitions>
    <EntityTypes>
      <ClassTypes>
        <ClassType ID="SQL.Server.Computer" Base="Windows!Microsoft.Windows.Server.Computer" Accessibility="Public" Abstract="false"></ClassType>
        <ClassType ID="AD.Server.Computer" Base="Windows!Microsoft.Windows.Server.Computer" Accessibility="Public" Abstract="false"></ClassType>
      </ClassTypes>
    </EntityTypes>
    <ModuleTypes>
      <DataSourceModuleType ID="Computer.Name.RegEx.Discovery.DS" Accessibility="Public">
        <Configuration>
          <xsd:element name="ClassId" type="xsd:string" minOccurs="0" maxOccurs="1" xmlns:xsd="http://www.w3.org/2001/XMLSchema" />
          <xsd:element name="NetBiosName" type="xsd:string" minOccurs="0" maxOccurs="1" xmlns:xsd="http://www.w3.org/2001/XMLSchema" />
          <xsd:element name="RegExPattern" type="xsd:string" minOccurs="0" maxOccurs="1" xmlns:xsd="http://www.w3.org/2001/XMLSchema" />
          <xsd:element name="IntervalHours" type="xsd:int" minOccurs="0" maxOccurs="1" xmlns:xsd="http://www.w3.org/2001/XMLSchema" />
        </Configuration>
        <ModuleImplementation>
          <Composite>
            <MemberModules>
              <DataSource ID="DS" TypeID="System!System.Discovery.Scheduler">
                <Scheduler>
                  <SimpleReccuringSchedule>
                    <Interval Unit="Hours">$Config/IntervalHours$</Interval>
                  </SimpleReccuringSchedule>
                  <ExcludeDates />
                </Scheduler>
              </DataSource>
              <ConditionDetection ID="CD" TypeID="System!System.ExpressionFilter">
                <Expression>
                  <RegExExpression>
                    <ValueExpression>
                      <Value Type="String">$Config/NetBiosName$</Value>
                    </ValueExpression>
                    <Operator>MatchesRegularExpression</Operator>
                    <Pattern>$Config/RegExPattern$</Pattern>
                  </RegExExpression>
                </Expression>
              </ConditionDetection>
              <ConditionDetection ID="Mapper" TypeID="System!System.Discovery.ClassSnapshotDataMapper">
                <ClassId>$Config/ClassId$</ClassId>
                <InstanceSettings>
                  <Settings>
                    <Setting>
                      <Name>$MPElement[Name="Windows!Microsoft.Windows.Computer"]/PrincipalName$</Name>
                      <Value>$Target/Property[Type="Windows!Microsoft.Windows.Computer"]/PrincipalName$</Value>
                    </Setting>
                    <Setting>
                      <Name>$MPElement[Name="System!System.Entity"]/DisplayName$</Name>
                      <Value>$Target/Property[Type="Windows!Microsoft.Windows.Computer"]/PrincipalName$</Value>
                    </Setting>
                  </Settings>
                </InstanceSettings>
              </ConditionDetection>
            </MemberModules>
            <Composition>
              <Node ID="Mapper">
                <Node ID="CD">
                  <Node ID="DS" />
                </Node>
              </Node>
            </Composition>
          </Composite>
        </ModuleImplementation>
        <OutputType>System!System.Discovery.Data</OutputType>
      </DataSourceModuleType>
    </ModuleTypes>
  </TypeDefinitions>
  <Monitoring>
    <Discoveries>
      <Discovery ID="SQL.Server.Computer.Discovery" Target="Windows!Microsoft.Windows.Server.Computer" Enabled="true">
        <Category>Discovery</Category>
        <DiscoveryTypes>
          <DiscoveryClass TypeID="SQL.Server.Computer" />
        </DiscoveryTypes>
        <DataSource ID="DS" TypeID="Computer.Name.RegEx.Discovery.DS">
          <ClassId>$MPElement[Name="SQL.Server.Computer"]$</ClassId>
          <NetBiosName>$Target/Property[Type="Windows!Microsoft.Windows.Computer"]/NetbiosComputerName$</NetBiosName>
          <RegExPattern>SQL-[a-z]{4}[0-9]{2}</RegExPattern>
          <IntervalHours>12</IntervalHours>
        </DataSource>
      </Discovery>
      <Discovery ID="AD.Server.Computer.Discovery" Target="Windows!Microsoft.Windows.Server.Computer" Enabled="true">
        <Category>Discovery</Category>
        <DiscoveryTypes>
          <DiscoveryClass TypeID="AD.Server.Computer" />
        </DiscoveryTypes>
        <DataSource ID="DS" TypeID="Computer.Name.RegEx.Discovery.DS">
          <ClassId>$MPElement[Name="AD.Server.Computer"]$</ClassId>
          <NetBiosName>$Target/Property[Type="Windows!Microsoft.Windows.Computer"]/NetbiosComputerName$</NetBiosName>
          <RegExPattern>DC[0-9]{2}</RegExPattern>
          <IntervalHours>12</IntervalHours>
        </DataSource>
      </Discovery>
    </Discoveries>
  </Monitoring>
  <LanguagePacks>
    <LanguagePack ID="ENU" IsDefault="true">
      <DisplayStrings>
        <DisplayString ElementID="Computer.Name.Example.MP">
          <Name>Computer Name Example MP</Name>
          <Description>Example for populating a custom class by using computer name RegEx patterns</Description>
        </DisplayString>
        <DisplayString ElementID="SQL.Server.Computer">
          <Name>SQL Server Computer</Name>
          <Description>Seed class representing a Windows Server computer with SQL installed</Description>
        </DisplayString>
        <DisplayString ElementID="SQL.Server.Computer.Discovery">
          <Name>SQL Server Computer Seed Discovery</Name>
          <Description>Discovery that populates the SQL Server Computer seed class based on computer name RegEx patterns</Description>
        </DisplayString>
        <DisplayString ElementID="AD.Server.Computer">
          <Name>AD Server Computer</Name>
          <Description>Seed class representing a Windows Server computer with SQL installed</Description>
        </DisplayString>
        <DisplayString ElementID="AD.Server.Computer.Discovery">
          <Name>AD Server Computer Seed Discovery</Name>
          <Description>Discovery that populates the AD Server Computer seed class based on computer name RegEx patterns</Description>
        </DisplayString>
      </DisplayStrings>
      <KnowledgeArticles></KnowledgeArticles>
    </LanguagePack>
  </LanguagePacks>
</ManagementPack>
