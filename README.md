<%@ page contentType= "text/html"
		 language   = "java"
         import     = "java.lang.String,
                       java.util.ArrayList, 
                       java.util.Vector, 
                       gov.fema.common.misc.UserDataContext,
                       gov.fema.common.util.FemaConstants,
                       com.jsptags.navigation.pager.IPagerConstants,
                       gov.fema.common.logging.LogUtil,
                       gov.fema.environmental.businterface.IEGrantsConstants,
                       gov.fema.environmental.businterface.IEHPConstants,
                       gov.fema.environmental.businterface.IQueueCodes,
                       gov.fema.environmental.util.NullChecker,
					   gov.fema.environmental.util.EHPUtility,
                       gov.fema.environmental.businterface.IEnvironmentalConstants,
                       gov.fema.environmental.businterface.IEnvironmentalDatabaseConstants,
                       gov.fema.environmental.businterface.ICaseManagementConstants,
                       gov.fema.environmental.businterface.IInboxConstants,
                       gov.fema.environmental.businterface.IInboxMessages,
                       gov.fema.environmental.valueobject.IInboxValueObject,
                       gov.fema.environmental.valueobject.UserInformationValueObject,
                       gov.fema.environmental.valueobject.IEGrantUserValueObject,
                       gov.fema.environmental.valueobject.IFilterListValueObject,
                       gov.fema.environmental.businterface.IReportsConstants,
                       gov.fema.environmental.valueobject.CaseMgtSelectProjectValueObject,
                       gov.fema.environmental.valueobject.ProgramInformationValueObject,
                       gov.fema.environmental.valueobject.IProgramInformationValueObject"
%>

<%-- Tag Library Definitions --%>
<%@ taglib uri="../../WEB-INF/tlds/pager-taglib.tld" prefix="pg" %>



<%     
    LogUtil logUtil = LogUtil.getLogger("gov.fema.inboxjsp");
    logUtil.debug("--------------------------------");
    logUtil.debug("inbox.jsp");

    Vector                         inboxEntries  = null;
    Integer                        totalRows     = null;
    IInboxValueObject              inboxEntry    = null;
    IFilterListValueObject         filterListEntry = null;
    String                         currentUser   = null;
    String                         programId     = null;
    String                         programName   = null;
    Vector                         errorMessages = null;
    boolean                        pendingWork   = false;
    boolean	showDeleteFilter	= false;
    
    String                         typeCode      = null; 

   
    String projectlistURL =    request.getContextPath()+"/VIEW_CASE_MANAGEMENT.do?actionType=LOAD_PROJECTS";
    UserDataContext userDataContext = (UserDataContext)session.getAttribute(FemaConstants.USER_DATA_CONTEXT_KEY);
    
    // Retrieve the username of the currently logged in user.
    IEGrantUserValueObject eGrantUser      = (IEGrantUserValueObject)userDataContext.getObject(IEGrantsConstants.EGRANT_USER);

    // Retrieve the sessionId from the currently logged-in user object.
    String sessionId     = ((IEGrantUserValueObject)userDataContext.getObject(IEGrantsConstants.EGRANT_USER)).getSessionId();

    currentUser = ((IEGrantUserValueObject)userDataContext.getObject(IEGrantsConstants.EGRANT_USER)).getUsername();
	
	Vector fiscalYears = null,  allDHSComponents = null, allDisasterNumbers = null, allStatesForUser =null, allRegionsForUser = null,allProgramsDropDown=null;
    String fiscalYear = null, program = null, component = null, disaster = null, stateCode = null, region = null,checkedOutBy = null;
    ProgramInformationValueObject progInfoValueObject = null;
    UserInformationValueObject userInformationValueObject = null;
   
    CaseMgtSelectProjectValueObject caseMgtSelectProjectValueObject=new CaseMgtSelectProjectValueObject();

    // Retrieve the fiscal years list from the seesion object.
    fiscalYears = (Vector)session.getAttribute(IReportsConstants.FISCAL_YEARS);
    allProgramsDropDown = (Vector)session.getAttribute(IEGrantsConstants.ALL_PROGRAMS_DROP_DOWN_LIST);
    allDHSComponents = (Vector)session.getAttribute(IEGrantsConstants.ALL_DHS_COMPONENTS);
    allDisasterNumbers = (Vector)session.getAttribute(IEGrantsConstants.ALL_DISASTER_EVENTS);
    allRegionsForUser = (Vector)session.getAttribute(IEGrantsConstants.REGION_NUMBERS);
    allStatesForUser = (Vector)session.getAttribute(IEGrantsConstants.STATE_CODES);
   	ArrayList projectIds=new ArrayList(); 
    
    if(session.getAttribute(IInboxConstants.SELECTED_FISCAL_YEAR)!=null)
    {
      fiscalYear = (String)session.getAttribute(IInboxConstants.SELECTED_FISCAL_YEAR);
    }
    
    if(session.getAttribute(IInboxConstants.SELECTED_PROGRAM)!=null)
    {
      program = (String)session.getAttribute(IInboxConstants.SELECTED_PROGRAM);
    }

    if(session.getAttribute(IInboxConstants.SELECTED_COMPONENT)!=null)
    {
      component = (String)session.getAttribute(IInboxConstants.SELECTED_COMPONENT);
    }
    
    if(session.getAttribute(IInboxConstants.SELECTED_DISASTER_NUMBER)!=null)
    {
      disaster = (String)session.getAttribute(IInboxConstants.SELECTED_DISASTER_NUMBER);
    }
    
    if(session.getAttribute(IInboxConstants.SELECTED_REGION_NR)!=null)
    {
      region = (String)session.getAttribute(IInboxConstants.SELECTED_REGION_NR);
    }
    
    if(session.getAttribute(IInboxConstants.SELECTED_STATE_CD)!=null)
    {
      stateCode = (String)session.getAttribute(IInboxConstants.SELECTED_STATE_CD);
      
    }
    
    if( session.getAttribute(IInboxConstants.SELECTED_CHECKED_OUT_BY)!=null )
    {
      checkedOutBy = (String)session.getAttribute(IInboxConstants.SELECTED_CHECKED_OUT_BY);
    }
    if( session.getAttribute(ICaseManagementConstants.PROJECTIDS)!=null )
    {
      projectIds = (ArrayList)session.getAttribute(ICaseManagementConstants.PROJECTIDS);
    }
    
    logUtil.debug("    currentUser: " + currentUser);
    logUtil.debug("    pendingWork: " + pendingWork);

    
    
    String sortInboxURL = request.getContextPath();
    
    String param =null,value=null;
    StringBuffer urlappender = new StringBuffer();
    String displayproject;
   
%>
	<link rel="stylesheet" type="text/css" href="css/easyui.css">
	<link rel="stylesheet" type="text/css" href="css/icon.css">
	<link rel="stylesheet" type="text/css" href="css/demo.css">
	<script type="text/javascript" src="javascript/jquery.min.js"></script>
	<script type="text/javascript" src="javascript/jquery.easyui.min.js"></script>
	
    <form id="ff" name="case" method="post">
     <input type="hidden" id="selectedprojectid" name="selectedprojectid" value="" />
     <input type="hidden" id="mode" name="mode" value="" />         
     <table border=1 style="width:80%;border-width: 1px;border-style: solid;border-color: rgb(149, 184, 231)" id="filterTable">
            <tr>
                <td>
                    <table class="noborder">
                      <tr>
                          <td class="labelrb" align="right"><label for="<%=IEGrantsConstants.COMPONENT_ID%>">DHS Component:&nbsp;&nbsp;</label></td>
                          <td>   
                              <select name="<%=IEGrantsConstants.COMPONENT_ID%>" id="<%=IEGrantsConstants.COMPONENT_ID%>" class="dropdown" >  
								<option></option>
								 <% 
                                if(allDHSComponents!=null)
                                {
                                    for (int i = 0; i < allDHSComponents.size(); i++) 
                                    { 
                                %> 
                                  <option  value="<%=allDHSComponents.elementAt(i)%>" <% if(component!=null && component.equalsIgnoreCase((String)allDHSComponents.elementAt(i))) { %> selected <% } %>>
                                      <%=allDHSComponents.elementAt(i) %>
                                  </option>
                                <%    
                                    }
                                }
                                %>
                               </select>
                              &nbsp;&nbsp;
                          </td>
            
                          <td class="labelrb" align="right"><label for="<%=IInboxConstants.SELECTED_PROGRAM%>">Program:&nbsp;&nbsp;</label></td>
                          <td>
                	          <select name="<%=IInboxConstants.SELECTED_PROGRAM%>" id="<%=IInboxConstants.SELECTED_PROGRAM%>" class="dropdown"  onchange="javascript:setActionAndSubmit('inbox','<%=request.getContextPath()%>/CHECK_FOR_DISASTER_PROGRAM.do?actionType=CHECK_FOR_DISASTER_PROGRAM')">
                                  <option></option>
                                   <%
                                if(allProgramsDropDown!=null)
                                {
                                    for(int j=0;j<allProgramsDropDown.size();j++)
                                    {
                                        progInfoValueObject = (ProgramInformationValueObject)allProgramsDropDown.get(j);
                                %>
                                  <option value="<%=EHPUtility.EHPEscape(progInfoValueObject.getProgramId())%>" <% if(program!=null && EHPUtility.EHPEscape(program).equalsIgnoreCase(EHPUtility.EHPEscape(progInfoValueObject.getProgramId()))) { %>selected <% } %>>
                      		          <%= EHPUtility.EHPEscape(progInfoValueObject.getProgramId()) %>
                                  </option>
                                <%  
                                    }
                                } 
                                %>
                              </select>
            		          &nbsp;&nbsp;
            		      </td>
             
            		      <td class="labelrb" align="right"><label for=="<%=IEGrantsConstants.DISASTER_ID%>">Disaster Number:&nbsp;&nbsp;</label></td>
            		      <td>   
            		          <select name="<%=IEGrantsConstants.DISASTER_ID%>" id="<%=IEGrantsConstants.DISASTER_ID%>" class="dropdown" >
                                  <option></option>
                                   <% 
                                if(allDisasterNumbers!=null)
                                {
                                    for (int i = 0; i < allDisasterNumbers.size(); i++) 
                                    { 
                                        if( allDisasterNumbers.elementAt(i)!=null && !((String)allDisasterNumbers.elementAt(i)).equalsIgnoreCase("null") )
                                        {
                                %> 
                		          <option  value="<%=allDisasterNumbers.elementAt(i)%>" <% if(disaster!=null && disaster.equalsIgnoreCase((String)allDisasterNumbers.elementAt(i))) { %> selected <% } %>>
                		              <%=allDisasterNumbers.elementAt(i) %>
                		          </option>
                                <%    
                                        }
                                    }
                                }
                                %>
                              </select>
                              &nbsp;&nbsp;
            		      </td>
               	      </tr>
        
        	          <tr>
                          <td class="labelrb" align="right"><label for="<%=IEGrantsConstants.REGION_NR%>">Region:&nbsp;&nbsp;</label></td>
            	          <td>
                              <select name="<%=IEGrantsConstants.REGION_NR%>" id="<%=IEGrantsConstants.REGION_NR%>" class="dropdown" onchange="javascript:setActionAndSubmit('inbox','<%=request.getContextPath()%>/FILTER_STATES_BY_REGION.do?actionType=FILTER_STATES_BY_REGION')">
                                  <option></option>
                                <% 
                                if(allRegionsForUser!=null)
                                {
                                    for (int i = 0; i < allRegionsForUser.size(); i++) 
                                    { 
                                      if(((String)allRegionsForUser.elementAt(i)).startsWith("H") && session.getAttribute(IInboxConstants.DISASTER_DROP_DOWN_ENABLED)!=null && (((String)session.getAttribute(IInboxConstants.DISASTER_DROP_DOWN_ENABLED)).equalsIgnoreCase("YES")) &&  ( program != null && (program.trim()).length() > 0 ))
                                      { continue; }
                                %> 
                                  <option  value="<%=allRegionsForUser.elementAt(i)%>" <%if(region!=null && region.equalsIgnoreCase((String)allRegionsForUser.elementAt(i))) { %> selected <% } %>>
                	                  <%=allRegionsForUser.elementAt(i) %>
                	              </option>
                                <%    
                                    } 
                                }
                                %>
                              </select>
              		          &nbsp;&nbsp;
                          </td>
                      
            		      <td class="labelrb" align="right"><label for="<%=IEGrantsConstants.STATE_CD%>">State:&nbsp;&nbsp;</label></td>
            		      <td>
            	    	      <select name="<%=IEGrantsConstants.STATE_CD%>" id="<%=IEGrantsConstants.STATE_CD%>" class="dropdown">  
                                  <option></option>
                                 <% 
                                if(allStatesForUser!=null)
                                {
                                    for (int i = 0; i < allStatesForUser.size(); i++) 
                                    {    
                                %> 
                                  <option  value="<%=allStatesForUser.elementAt(i)%>" <% if(stateCode!=null && stateCode.equalsIgnoreCase((String)allStatesForUser.elementAt(i))) { %> selected <% } %>>
                	                  <%=allStatesForUser.elementAt(i) %>
                	              </option>
                                <%    
                                    }
                                }
                                %>
              		          </select>
              		          &nbsp;&nbsp;
                          </td>
            
                          <td class="labelrb" align="right"><label for="<%=IReportsConstants.FISCAL_YEAR%>">FY:&nbsp;&nbsp;</label></td>
            		      <td> 
                              <select name="<%=IReportsConstants.FISCAL_YEAR%>" id="<%=IReportsConstants.FISCAL_YEAR%>" class="dropdown">
                                  <option></option>
                                  <% 
                                if(fiscalYears!=null)
                                {
                                    for (int i = 0; i < fiscalYears.size(); i++) 
                                    { 
                                %> 
                                  <option  value="<%=fiscalYears.elementAt(i)%>" <% if(fiscalYear!=null && fiscalYear.equalsIgnoreCase((String)fiscalYears.elementAt(i))) { %> selected <% } %>>
                                      <%=fiscalYears.elementAt(i) %>
                                  </option>
                                <%    
                                    }
                                }
                                %>
                              </select>
             		    	  &nbsp;&nbsp;
            		      </td>
						   <td> <button class="filterbutton" onclick="this.disabled=true;setActionAndSubmit('ff','<%=projectlistURL%>')">Search</button> </td>  
 	      	          </tr>
                      <tr>
     
                          <td> &nbsp;&nbsp; </td>
            		      <td> &nbsp;&nbsp; </td>     
             		      
             		      <td class="labelrb" align="right"><label for="<%=ICaseManagementConstants.PROJECT_ID_DROPDOWN%>">Select Project ID:&nbsp;&nbsp;</label></td>
            		      <td>
            	    	      <select name="<%=ICaseManagementConstants.PROJECT_ID_DROPDOWN%>" id="<%=ICaseManagementConstants.PROJECT_ID_DROPDOWN%>" class="dropdown">()   
                                  <option></option>
                                   <% 
                                if(projectIds!=null)
                                {
                                    for (int i = 0; i < projectIds.size(); i++) 
                                    { 
                                    caseMgtSelectProjectValueObject=(CaseMgtSelectProjectValueObject) projectIds.get(i);
                                    if (caseMgtSelectProjectValueObject.getEHPId()>0) 
                                    {
                                    	displayproject=caseMgtSelectProjectValueObject.getProjectId();
                                    	
                                    }
                                    else
                                    {
                                    	displayproject=caseMgtSelectProjectValueObject.getProjectId()+"*";
                                    }
                                %> 
                                  <option  value="<%=caseMgtSelectProjectValueObject.getCaseMgtId()+"|"+caseMgtSelectProjectValueObject.getEHPId()%>">
                                      <%=displayproject%>
                                  </option>
                                <%    
                                    }
                                }
                                %>
                               </select>
              		          &nbsp;&nbsp;
            		      </td>
						<td> <button class="filterbutton" type="button" id="new" onclick="newClick()">New</button> </td>
            		     
 	      	          </tr>
                    
		          </table> 	
              </td>
            </tr>
        </table>	
      </form>
<br>
<div id="maintab" class="easyui-tabs" tabWidth="95px" tabHeight="50px"  style="width:1150px">
	
	<div title="&nbsp;&nbsp;&nbsp;Project &nbsp;&nbsp;&nbsp; Details" style="padding:2px">
			  <form id="ff1" name="case" method="post"> 
				<table style="padding:0px;width:1100px" >
				<tr>
				<td>
					<table tyle="padding:10px">
					<tr>
						<td style="padding-TOP:5px;">
						<label  class="labelrb">Project Id:</label>
						</td>
						<td style="padding-RIGHT:20px">
						<input id="projectId" name="projectId" type=text>
						</td>
					</tr>
					<tr>
					</tr>
					<tr>
						<td style="padding-TOP:5px;">
						<label  class="labelrb">Program:</label>
						</td>
						<td style="padding-RIGHT:20px">
						<input id="programCode" name="programCode" type=text>
						</td>
						<td style="padding-TOP:5px">
						<label class="labelrb">DR Number:</label>
						</td>
						<td style="padding-RIGHT:20px">
						<input id="disasterEventId" name="disasterEventId" type=text>
						</td>
						<td style="padding-TOP:5px">
						<label class="labelrb">Subgrantee/Grantee:</label>
						</td>
						<td style="padding-RIGHT:20px">
						<input id="Grantee" name="Grantee" type=text>
						</td>
						<td style="padding-TOP:5px">
						<label class="labelrb">State:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input id="state" name="state" type=text>
						</select>
						</td>
						<td style="padding-TOP:5px">
						<label class="labelrb">County:</label>
						</td>
						<td>
						<input type=text name="county">
						</td>
					</tr>
					</table>
					<BR>
				</td>
				</tr>
				<tr>
					<td>
					<table>
					<tr>
						<td style="padding-TOP:5px">
						<label class="labelrb">Amount:</label>
						</td>
						<td style="padding-RIGHT:20px">
						<input name="amount" class="easyui-numberbox"  data-options="precision:0,groupSeparator:',',decimalSeparator:'.',prefix:'$'"></input>
						</td>
						<td style="padding-TOP:5px">
						<label class="labelrb">*Category (PA Only):</label>
						</td>
						<td style="padding-RIGHT:20px">
						<select name="paCategory" id="paCategory" class="dropdown" style="width:100px;">   
		                     <option value="A">A</option>
		                     <option value="B">B</option>
		                     <option value="C">C</option>
		                     <option value="D">D</option>
		                     <option value="E">E</option>
		                     <option value="F">F</option>
		                     <option value="G">G</option>
		                     <option value="H">H</option>
						</select>
						</td>
						<td style="padding-TOP:5px">
						<label class="labelrb">*Nepa Level:</label>
						</td>
						<td style="padding-RIGHT:20px">
						 <select name="nepaLevel" id="nepaLevel" class="dropdown" style="width:100px;">   
		                        <option value="CATEX">CATEX</option>
						</select>
						</td>
						<td style="padding-TOP:5px">
						<label class="labelrb">Priority:</label>
						</td>
						<td style="padding-RIGHT:20px">
						 <select name="priority" id="priority" class="dropdown" style="width:100px;">   
		                	<option value="H">HIGH</option>
							<option value="M">MEDIUM</option>
							<option value="L">LOW</option>
						</select>
						</td>
						</tr>
						</table>
						<BR>
						</td>
				<tr>
					<td >
					<table>
						<tr>
						<td  style="padding-TOP:5px">
						<label class="labelrb">Triage Date:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input name="triageDate" class="easyui-datebox" editable="false"></input>
						</td>
						<td  style="padding-TOP:5px">
						<label class="labelrb">Estimated Review Completion Date:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input name="estReviewCmplDate" class="easyui-datebox" editable="false"></input>
						</td>
						<td  style="padding-TOP:5px">
						<label class="labelrb">Adjusted Review Completion Date:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input name="adjReviewCmplDate" class="easyui-datebox" editable="false"></input>
						</td>
						</tr>
					</table>
					<BR>
					</td>
					</tr>
					<tr>
					<td>
					<table>
					<tr>
						<td  style="padding-TOP:5px">
						<label class="labelrb">Date in NEMIS:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input name="inNemisDate" class="easyui-datebox" editable="false"></input>
						</td>
						<td  style="padding-TOP:5px">
						<label  class="labelrb">Date Out of NEMIS:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input name="outNemisDate" class="easyui-datebox" editable="false"></input>
						</td>
					</tr>
					</table>
					<BR>
					</td>
				</tr>

				<tr>

						<td>
						<table>
						<tr>
						<td  style="padding-TOP:5px">
						<label class="labelrb">Statement of Work:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input name="sow" class="easyui-textbox" data-options="multiline:true" value="" style="width:300px;height:90px">
						</td>
						<td  style="padding-TOP:5px">
						<label class="labelrb">Quick Status:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<textarea name="quickStatusString" rows="5" cols="50">
						</textarea>
						</td>
						<td  style="padding-TOP:5px">
						<label class="labelrb">Reference Project ID:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<label><input name="refprojectid" type=text></label>
						</td>
						</tr>
						</table>
							<BR>
						</td>
				</tr>
				<tr>
						<td>
						<table>
						<tr>
						
						<td  style="padding-TOP:5px">
						<label class="labelrb">Special Issues:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input name="specialIssues" class="easyui-textbox" data-options="multiline:true" value="" style="width:300px;height:90px">
						</td>
						<td  style="padding-TOP:5px">
						<label class="labelrb">Comments:</label>
						</td>
						<td  style="padding-RIGHT:20px">
						<input name="comment" class="easyui-textbox" data-options="multiline:true" value="" style="width:300px;height:90px">
						</td>
						<td>
						</td>
						<td  style="padding-TOP:5px">
						 
						</td>
						</tr>
						</table>
						<br>
						</td>
				</tr>
				<tr>
				<td colspan=5 align="right"> <button class="filterbutton" type="button" id="saveform1" onclick="saveform(this)">Save</button> </td>  
				</tr>
				</table>
				  <input type="hidden" id="caseMgtId" name="caseMgtId" value="" />
				  <input type="hidden" id="EHPId" name="EHPId" value="" />
				  <input type="hidden" id="ehpReviewStartDatea" name="ehpReviewStartDate" value="" />
				  <input type="hidden" id="ehpReviewEndDatea" name="ehpReviewEndDate" value="" />      
				  <input type="hidden" id="reviewersa" name="reviewers" value="" />  
				  <input type="hidden" id="occSentDatea" name="occSentDate" value="" />
				  <input type="hidden" id="occApprovedDatea" name="ehpReviewEndDate" value="" />  
				  <input type="hidden" id="reoSentdatea" name="ehpReviewEndDate" value="" />  
				  <input type="hidden" id="reoApprovedDatea" name="ehpReviewEndDate" value="" />  
				  <input type="hidden" id="ff1loadstatus" name="ff1loadstatus" value="" />  
				  <input type="hidden" id="ff1message" name="ff1message" value="" />  
				</form>
	</div>
	<div title="EHP Review/ FONSI" style="padding:10px">
	<form id="ff2" name="ff2" method="post"> 
		<table border=0>
					<tr>
					<td>
						<table  style="border-width: 1px;border-style: solid;border-color: rgb(149, 184, 231">
							<tr>
								<td class="labelrb" style="text-align:left;font-weight: bold;">
								EHP Review
								</td>
								
							</tr>
							<br>
							<tr>
								<td class="labelrb" style="padding-BOTTOM:5px">
								Date EHP Review Started:
								</td>
								<td>
								<input name="ehpReviewStartDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
								</td>
								
								<td class="labelrb" style="padding-LEFT:20px;padding-BOTTOM:5px">
								Date EHP Review Completed:
								</td>
								<td>
								<input name="ehpReviewEndDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
								</td>
								
								
								
							</tr>
							<tr>
							<td class="labelrb" style="padding-LEFT:20px;vertical-align:top">
								EHP Reviewer:
								</td>
								<td>
								<label><input name="reviewers" class="easyui-textbox" data-options="multiline:true" value="" style="width:300px;height:90px"></label>
								</td>
							</tr>
							
						</table>
					</td>
					</tr>
					<tr>
					</tr>
					<tr>
					</tr>
					<tr>
					</tr>
					<tr>
					</tr>
				
					<tr>
					<td>
					<table style="border-width: 1px;border-style: solid;border-color: rgb(149, 184, 231">
					<tr>
					<td class="labelrb" colspan=4 style="text-align:left;font-weight: bold;">
					FONSI
					<br>
					</td>
					</tr>
					<tr>
					<td class="labelrb" style="padding-BOTTOM:5px">
					Date Sent to OCC:
					</td>
					<td>
					<input name="occSentDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
					</td>
					<td class="labelrb" style="padding-LEFT:20px;padding-BOTTOM:5px">
					Date Approved by OCC:
					</td>
					<td>
					<input name="occApprovedDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
					</td>
					<td class="labelrb" style="padding-LEFT:20px;padding-BOTTOM:5px">
					Date Sent to REO:
					</td>
					<td>
					<input name="reoSentdate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
					</td>
					<td class="labelrb" style="padding-LEFT:20px;padding-BOTTOM:5px;">
					Date REO Approved:
					</td>
					<td>
					<input name="reoApprovedDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
					</td>
					</tr>
					</table>
					</td>
					</tr>
					<tr rowspan=4>
					</tr>
					<tr>
					<td colspan=5 align="right"> <button class="filterbutton" type="button" id="saveform2" onclick="saveform(this)">Save</button> </td>  
					</tr>
					
					</table>
					</form>
				
	</div>
	<div title="RFI" style="padding:10px">
	
				<table id="dg_rfi" title="" toolbar="#toolbarRFI" pagination="false"
				rownumbers="false" singleSelect="true" nowrap="false"> 
				<thead>
				<tr>
					<th field="rfiSendDate" width="100">Date RFI Sent</th>
					<th field="rfiReturnedDate" width="100">Date RFI Returned</th>
					<th field="ProgramPOCSentTo" width="140">Program POC RFI Sent To</th>
					<th field="rfiNotes" width="400">Reason for RFI/RFI Notes</th>
					<th field="rfiId" width="50" hidden="true">rfiid</th>
				</tr>
				</thead>
				</table>

				<div id="toolbarRFI">
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newRFIStatus()">New</a>
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editRFIStatus()">Edit</a>
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removeRFIStatus()">Delete</a>
				</div>
				<div id="rfidlg" class="easyui-dialog" style="width:600px;height:280px;padding:10px 20px" closed="true" buttons="#rfidlg-buttons" data-options="modal:true">
					<form id="fmrfi" method="post" novalidate> 
					<div class="fitem" style="padding:5px 0px">
					<label  style="font-weight:bold;">Date RFI Sent:</label>
					<input name="rfiSendDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
					</div>
					<div class="fitem" style="padding:5px 0px">
					<label  style="font-weight:bold;">Date RFI Returned:</label>
					<input name="rfiReturnedDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
					</div>
					<div class="fitem" style="padding:5px 0px">
					<label  style="font-weight:bold;">Program POC RFI Sent to:</label>
						<input name="ProgramPOCSentTo" class="easyui-textbox" style="width: 116px; height: 20px;" validType="length[0,50]">
					</div>
					<div class="fitem" style="padding:5px 0px">
					<label style="font-weight:bold;">Reason for RFI/RFI Notes:</label>
					<input name="rfiNotes" class="easyui-textbox" data-options="multiline:true" value="" style="width:300px;height:80px" validType="length[0,500]">
					</div>
					 <input type="hidden" id="rfiId" name="rfiId" value="0" />  
					</form>	
				</div>
				
				<div id="rfidlg-buttons">
				<a href="javascript:void(0)" class="easyui-linkbutton c6"  id="rfisave" onclick="saveform(this)" style="width:90px">Save</a>
				<a href="javascript:void(0)" class="easyui-linkbutton" onclick="javascript:$('#rfidlg').dialog('close')" style="width:90px">Cancel</a>
				</div>
					
	</div>
	<div title="Status" style="padding:10px">
								<table id="dgstatus" title="" style="auto"
								toolbar="#statustoolbar" pagination="false"
								rownumbers="false" fitColumns="false" singleSelect="true">
								<thead>
								<tr>
								<th field="status" width="100">Status</th>
								<th field="statusStartDate" width="100">Start Date</th>
								<th field="statusEndDate" width="100">End Date</th>
								<th field="statusId" width="100" hidden="true">statusid</th>
								</tr>
								</thead>
								</table>
								<div id="statustoolbar">
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newstatus()">New</a>
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editstatus()">Edit</a>
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removestatus()">Delete</a>
								</div>
								<div id="dlgstatus" class="easyui-dialog" style="width:600px;height:280px;padding:10px 20px" closed="true" data-options="modal:true" buttons="#dlg-buttons">
									<form id="fmstatus" method="post" novalidate> 
									<div class="fitem" style="padding:5px 0px">
									<label style="font-weight:bold;">Status:</label>
									<select class="easyui-combobox" data-options="editable:false" name="status" style="width:100px;">
									<option value="Completed">Completed</option>
									<option value="Pending">Pending</option>
									</select>
									</div>
									<div class="fitem" style="padding:5px 0px">
									<label style="font-weight:bold;">Start Date:</label>
									<input name="statusStartDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
									</div>
									<div class="fitem" style="padding:5px 0px">
									<label style="font-weight:bold;">End Date:</label>
									<input name="statusEndDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
									</div>
									<input type="hidden" id="statusId" name="statusId" value="0" />  
									</form>
								</div>
								<div id="dlg-buttons">
								<a href="javascript:void(0)" class="easyui-linkbutton c6"  id="statussave" onclick="saveform(this)" style="width:90px">Save</a>
								<a href="javascript:void(0)" class="easyui-linkbutton"  onclick="javascript:$('#dlgstatus').dialog('close')" style="width:90px">Cancel</a>
								</div>
	</div>
	<div title="Consultation/ Coordination" style="padding:10px">
									<table id="dgconsult" title="" style="auto"
									toolbar="#consultoolbar" pagination="false"
									rownumbers="false" fitColumns="false" singleSelect="true">
									<thead>
									<tr>
									<th field="agency" width="100px">Agency</th>
									<th field="action" width="100px">Action</th>
									<th field="requestDate" width="100px">Request Date</th>
									<th field="consultVia" width="60px">Via</th>
									<th field="contact" width="100px">Contact</th>
									<th field="responseDueDate" width="105px">Response Due Date</th>
									<th field="responseReceivedDate" width="105px">Response Received Date</th>
									<th field="conditionsFlag" width="100px">Conditions Apply</th>
									<th data-options="field:'formalflag',align:'center',editor:{type:'checkbox',options:{on:'Y',off:''}}">Formal</th>
									<th data-options="field:'informalflag',align:'center',editor:{type:'checkbox',options:{on:'Y',off:''}}">Informal</th>
									<th data-options="field:'pendingflag',align:'center',editor:{type:'checkbox',options:{on:'Y',off:''}}" width="105px">Pending Consultation</th>
									<th data-options="field:'consultationId',align:'center',editor:{type:'checkbox',options:{on:'Y',off:''}}" width="100px" hidden="true"></th>
									</tr>
									</thead>
									</table>
									<div id="consultoolbar">
									<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newConsult()">New</a>
									<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editConsult()">Edit</a>
									<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removeConsult()">Delete</a>
									</div>
									<div id="dlgconsult" class="easyui-dialog" style="width:600px;height:520px;padding:10px 20px" closed="true" data-options="modal:true" buttons="#dlg-buttons">
									<form id="fmcons" method="post" novalidate>
										<div class="fitem" style="padding:5px 0px">
										<label>Agency:</label>
										<select class="easyui-combobox" data-options="editable:false" name="agency" style="width:100px;">
											<option value="NOAA">NOAA</option>
											<option value="State DNR">State DNR</option>
										</select>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Action:</label>
										<input name="action" class="easyui-textbox" style="width: 116px; height: 20px;">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Request Date:</label>
										<input name="requestDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false">></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Via:</label>
										<input name="consultVia" class="easyui-textbox" style="width: 116px; height: 20px;">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Contact:</label>
										<input name="contact" class="easyui-textbox" style="width: 116px; height: 20px;">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Response Due Date:</label>
										<input name="responseDueDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false" editable="false">>></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Response Received Date:</label>
										<input name="responseReceivedDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false">></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Conditions Apply:</label>
										<input type="checkbox" name="conditionsFlag" value="Y">
										<!--<input name="condond" class="easyui-textbox">-->
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Formal:</label>
										<input type="checkbox" name="formalflag" value="Y">
										<!--<input name="conformal" class="easyui-textbox">-->
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Informal:</label>
										<input type="checkbox" name="informalflag" value="Y">
										<!--<input name="coninformal" class="easyui-textbox">-->
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Pending Consultation:</label>
										<input type="checkbox" name="pendingflag" value="Y">
										</div>
										<input type="hidden" id="consultationId" name="consultationId" value="0" />  
									</form>
									</div>
									<div id="dlg-buttons">
									<a href="javascript:void(0)" class="easyui-linkbutton c6" id="consusave"  onclick="saveform(this)" style="width:90px">Save</a>
									<a href="javascript:void(0)" class="easyui-linkbutton"  onclick="javascript:$('#dlgconsult').dialog('close')" style="width:90px">Cancel</a>
									</div>
	</div>
	<div title="SHPO Consult" style="padding:10px">
									<table id="dgshpo" title="" style="auto"
									toolbar="#toolbarshpo" pagination="false"
									rownumbers="false" fitColumns="false" singleSelect="true">
										<thead>
										<tr>
										<th field="requestDate" width="100px">Request Date</th>
										<th field="consultVia" width="100px">Via</th>
										<th field="contact" width="200px">Contact</th>
										<th field="responseDueDate" width="105px">Response Due Date</th>
										<th field="responseReceivedDate" width="105px">Response Received Date</th>
										<th field="conditionsFlag" >Conditions Apply</th>
										<th field="pendingFlag" >Pending Consultation</th>
										<th field="shpoConsultId" width="100px" hidden="true"></th>
										</tr>
										</thead>
									</table>
									<div id="toolbarshpo">
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newShpo()">New</a>
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editShpo()">Edit</a>
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removeShpo()">Delete</a>
									</div>

									<div id="dlgshpo" class="easyui-dialog" style="width:600px;height:520px;padding:10px 20px" closed="true" data-options="modal:true" buttons="#dlg-buttons">
									<form id="fmshpo" method="post" novalidate>
										<div class="fitem" style="padding:5px 0px">
										<label>Request Date:</label>
										<input name="requestDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false">></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Via:</label>
										<input name="consultVia" class="easyui-textbox" style="width: 116px; height: 20px;">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Contact:</label>
										<input name="shpocontact" class="easyui-textbox" style="width: 116px; height: 20px;">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Response Due Date:</label>
										<input name="responseDueDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false">></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Response Recieved:</label>
										<input name="responseReceivedDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false">></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Conditions Apply:</label>
										<input type="checkbox" name="conditionsFlag" value="Y">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Pending Consultation:</label>
										<input type="checkbox" name="pendingFlag" value="Y">
										</div>
										<input type="hidden" id="shpoConsultId" name="shpoConsultId" value="0" />  
									</form>
									</div>
									<div id="dlg-buttons">
									<a href="javascript:void(0)" class="easyui-linkbutton c6" id="shposave"  onclick="saveform(this)" style="width:90px">Save</a>
									<a href="javascript:void(0)" class="easyui-linkbutton"  onclick="javascript:$('#dlgshpo').dialog('close')" style="width:90px">Cancel</a>
									</div>
	
	</div>
	<div title="Tribal Consult" style="padding:10px">
								<table id="dgtribal" title="" style="auto"
								toolbar="#toolbarthpo" pagination="false"
								rownumbers="false" singleSelect="true">
								<thead>
								<tr>
								<th field="tribe" width="175px">Tribe</th>
								<th field="requestDate" width="100px"> Request Date</th>
								<th field="consultVia" width="70PX">Via</th>
								<th field="contact" width="175px">Contact</th>
								<th field="followUpDate" width="100px">Follow up Date</th>
								<th field="responseDueDate" width="105px">Response Due Date</th>
								<th field="responseReceivedDate" width="105px">Response Received Date</th>
								<th field="conditionsFlag" width="100px">Conditions Apply</th>
								<th field="pendingFlag" width="105px">Pending Consultation</th>
								<th field="completeFlag" width="110px">Consultation Complete</th>
								<th field="thpoConsultId" width="100px" hidden="true"></th>
								</tr>
								</thead>
								</table>
								<div id="toolbarthpo">
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newTribal()">New</a>
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editTribal()">Edit</a>
								<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removeTribal()">Delete</a>
								</div>
								<div id="dlgtribal" class="easyui-dialog" style="width:600px;height:520px;padding:10px 20px" closed="true" data-options="modal:true" buttons="#dlg-buttons">
									<form id="fmtribal" method="post" novalidate>
										<div class="fitem" style="padding:5px 0px">
										<label>Tribe:</label>
										<input name="tribe" class="easyui-textbox" style="width: 116px; height: 20px;">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Request Date:</label>
										<input name="requestDate" class="easyui-datebox" style="width: 116px; height: 20px;"></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Via:</label>
										<input name="consultVia" class="easyui-textbox" style="width: 116px; height: 20px;">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Contact:</label>
										<input name="contact" class="easyui-textbox" style="width: 116px; height: 20px;">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Follow up Date:</label>
										<input name="followUpDate" class="easyui-datebox" style="width: 116px; height: 20px;"></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Response Due Date:</label>
										<input name="responseDueDate" class="easyui-datebox" style="width: 116px; height: 20px;"></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Response Received:</label>
										<input name="responseReceivedDate" class="easyui-datebox" style="width: 116px; height: 20px;"></input>
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Conditions Apply:</label>
										<input type="checkbox" name="conditionsFlag" value="Y">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Pending Consultation:</label>
										<input type="checkbox" name="pendingFlag" value="Y">
										</div>
										<div class="fitem" style="padding:5px 0px">
										<label style="font-weight:bold;">Consultation Complete:</label>
										<input type="checkbox" name="completeFlag" value="Y">
										</div>
										<input type="hidden" id="thpoConsultId" name="thpoConsultId" value="0" />  
									</form>
									</div>
									<div id="dlg-buttons">
									<a href="javascript:void(0)" class="easyui-linkbutton c6" id="tribalsave"  onclick="saveform(this)" style="width:90px">Save</a>
									<a href="javascript:void(0)" class="easyui-linkbutton"  onclick="javascript:$('#dlgtribal').dialog('close')" style="width:90px">Cancel</a>
									</div>
	</div>
	<div title="Internal Letter" style="padding:10px">
										<table id="dgintletter" title="" style="auto"
										toolbar="#toolbarint" pagination="false"
										rownumbers="false" singleSelect="true" nowrap="false">
										<thead>
											<tr>
											<th field="authorRequestDate"  width="106px">Author Request Date</th>
											<th field="authorCompleteDate"  width="110px">Author Completed Date</th>
											<th field="author" width="150px">Letter Author</th>
											<th field="reviewRequestDate"  width="110px">Review Request Date</th>
											<th field="reviewCompleteDate"  width="110px">Review Completed Date</th>
											<th field="letterReviewer"  width="150px">Letter Reviewer</th>
											<th field="notes" width="300px">Notes</th>
											<th field="letterId" width="100px" hidden="true"></th>
											</tr>
										</thead>
										</table>
										<div id="toolbarint">
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newLetter()">New</a>
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editLetter()">Edit</a>
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removeLetter()">Delete</a>
										</div>
							
										<div id="dlgintletter" class="easyui-dialog" style="width:400px;height:400px;padding:10px 20px" closed="true" data-options="modal:true" buttons="#dlg-buttons">
										<form id="fmintletter" method="post" novalidate>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Author Request Date:</label>
											<input name="authorRequestDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Author Completed Date:</label>
											<input name="authorCompleteDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Letter Author:</label>
											<input name="author" class="easyui-textbox" style="width: 116px; height: 20px;">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Review Request Date:</label>
											<input name="reviewRequestDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Review Completed Date:</label>
											<input name="reviewCompleteDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Letter Reviewer:</label>
											<input name="letterReviewer" class="easyui-textbox" style="width: 116px; height: 20px;">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Notes:</label>
											<input name="notes" class="easyui-textbox" data-options="multiline:true" value="" style="width:300px;height:80px">
											</div>
											<input type="hidden" id="letterId" name="letterId" value="0" />  
										</form>
										</div>
									<div id="dlg-buttons">
									<a href="javascript:void(0)" class="easyui-linkbutton c6" id="intlettersave"  onclick="saveform(this)" style="width:90px">Save</a>
									<a href="javascript:void(0)" class="easyui-linkbutton"  onclick="javascript:$('#dlgintletter').dialog('close')" style="width:90px">Cancel</a>
									</div>
							
							
	</div>
	<div title="Executive  Order" style="padding:10px">
										<table id="dgexorder" title="" style="auto"
										toolbar="#toolbarexec" pagination="false"
										rownumbers="false"  singleSelect="true">
										<thead>
										<tr>
										<th field="requestReceivedDate" width="105px">Request Received</th>
										<th field="agencyName" width="200px">Agency Name</th>
										<th field="requestUser" width="200px">By Whom</th>
										<th field="concurrenceCompleteDate" width="105px">Concurrence Completion Date</th>
										<th field="replySentFlag" width="105px">Reply Sent</th>
										<th field="execOrderId" width="50px" hidden="true"></th>
										</tr>
										</thead>
										</table>
										<div id="toolbarexec">
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newExorder()">New</a>
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editExorder()">Edit</a>
										<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removeExorder()">Delete</a>
										</div>
			
										<div id="dlgexorder" class="easyui-dialog" style="width:400px;height:400px;padding:10px 20px" closed="true" data-options="modal:true" buttons="#dlg-buttons">
										<form id="fmexorder" method="post" novalidate>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Request Received:</label>
											<input name="requestReceivedDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Agency Name:</label>
											<input name="agencyName" class="easyui-datebox" style="width: 116px; height: 20px;"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">By Whom:</label>
											<input name="requestUser" class="easyui-textbox" style="width: 116px; height: 20px;">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Concurrence Completion Date:</label>
											<input name="concurrenceCompleteDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Reply Sent:</label>
											<input type="checkbox" name="replySentFlag" value="Y">
											</div>
											<input type="hidden" id="execOrderId" name="execOrderId" value="0" />  
										</form>
										</div>
									<div id="dlg-buttons">
									<a href="javascript:void(0)" class="easyui-linkbutton c6" id="exordersave"  onclick="saveform(this)" style="width:90px">Save</a>
									<a href="javascript:void(0)" class="easyui-linkbutton"  onclick="javascript:$('#dlgexorder').dialog('close')" style="width:90px">Cancel</a>
									</div>
			
			
	</div>
	<div title="Allowances (Section 106)" style="padding:10px">
									<table id="dgallow106" title="" style="auto"
									toolbar="#toolbarallow" pagination="false"
									rownumbers="false" fitColumns="false" singleSelect="true">
									<thead>
									<tr>
									<th field="title" width="200">Allowance Title/Description</th>
									<th field="category" width="200">Category</th>
									<th field="effectiveDate" >Effective Date</th>
									<th field="expirationDate" >Expiration Date</th>
									<th field="notes" width="300">Notes</th>
									<th field="estCostSavings" >Estimated Cost Savings</th>
									<th field="estTimeSavings" >Estimated Time Savings</th>
									<th field="allowanceId" width="50px" hidden="true"></th>
									</tr>
									</thead>
									</table>
									<div id="toolbarallow">
									<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newallow106()">New</a>
									<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editallow106()">Edit</a>
									<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removenewallow106()">Delete</a>
									</div>
									
									<div id="dlgallow106" class="easyui-dialog" style="width:400px;height:400px;padding:10px 20px" closed="true" data-options="modal:true" buttons="#dlg-buttons">
										<form id="fmallow106" method="post" novalidate>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Allowance Title/Description:</label>
											<input name="title" class="easyui-textbox" style="width: 116px; height: 20px;">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Category:</label>
											<input name="category" class="easyui-textbox" style="width: 116px; height: 20px;">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Effective Date:</label>
											<input name="effectiveDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Expiration Date:</label>
											<input name="expirationDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Notes:</label>
											<input name="notes" class="easyui-textbox" data-options="multiline:true" value="" style="width:300px;height:80px">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Estimated Cost Savings:</label>
											<input name="estCostSavings" class="easyui-numberbox"  style="width:100px" data-options="precision:0,groupSeparator:',',decimalSeparator:'.',prefix:'$'"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Estimated Time Savings:</label>
											<input name="estTimeSavings" class="easyui-numberbox"   style="width:100px" data-options="precision:0,groupSeparator:',',decimalSeparator:'.',prefix:'$'"></input>
											</div>
											<input type="hidden" id="allowanceId" name="allowanceId" value="0" />  
										</form>
										</div>
									<div id="dlg-buttons">
									<a href="javascript:void(0)" class="easyui-linkbutton c6" id="allow106save"  onclick="saveform(this)" style="width:90px">Save</a>
									<a href="javascript:void(0)" class="easyui-linkbutton"  onclick="javascript:$('#dlgallow106').dialog('close')" style="width:90px">Cancel</a>
									</div>
	</div>
	<div title="Other Allowances" style="padding:10px">
										<table id="dgotherallow" title="" style="auto"
										toolbar="#toolbarotherallow" pagination="false"
										rownumbers="false" fitColumns="false" singleSelect="true">
										<thead>
										<tr>
										<th field="title" width="200">Allowance Title/Description</th>
										<th field="category" width="200">Category</th>
										<th field="effectiveDate" >Effective Date</th>
										<th field="expirationDate" >Expiration Date</th>
										<th field="notes" width="300">Notes</th>
										<th field="estCostSavings" >Estimated Cost Savings</th>
										<th field="estTimeSavings" >Estimated Time Savings</th>
										<th field="othallowanceId" width="50px" hidden="true"></th>
										</tr>
										</thead>
										</table>
										<div id="toolbarotherallow">
											<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-add" plain="true" onclick="newotherallow()">New</a>
											<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-edit" plain="true" onclick="editotherallow()">Edit</a>
											<a href="javascript:void(0)" class="easyui-linkbutton" iconCls="icon-remove" plain="true" onclick="removeotherallow()">Delete</a>
										</div>
										
										<div id="dlgotherallow" class="easyui-dialog" style="width:400px;height:400px;padding:10px 20px" closed="true" data-options="modal:true" buttons="#dlg-buttons">
										<form id="fmotherallow" method="post" novalidate>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Allowance Title/Description:</label>
											<input name="title" class="easyui-textbox" style="width: 116px; height: 20px;">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Category:</label>
											<input name="category" class="easyui-textbox" style="width: 116px; height: 20px;">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Effective Date:</label>
											<input name="effectiveDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Expiration Date:</label>
											<input name="expirationDate" class="easyui-datebox" style="width: 116px; height: 20px;" editable="false"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Notes:</label>
											<input name="notes" class="easyui-textbox" data-options="multiline:true" value="" style="width:300px;height:80px">
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Estimated Cost Savings:</label>
											<input name="estCostSavings" class="easyui-numberbox"  style="width:100px" data-options="precision:0,groupSeparator:',',decimalSeparator:'.',prefix:'$'"></input>
											</div>
											<div class="fitem" style="padding:5px 0px">
											<label style="font-weight:bold;">Estimated Time Savings:</label>
											<input name="estTimeSavings" class="easyui-numberbox"  style="width:100px" data-options="precision:0,groupSeparator:',',decimalSeparator:'.',prefix:'$'"></input>
											</div>
											<input type="hidden" id="othallowanceId" name="allowanceId" value="0" />  
										</form>
										</div>
									<div id="dlg-buttons">
									<a href="javascript:void(0)" class="easyui-linkbutton c6" id="otherallowsave"  onclick="saveform(this)" style="width:90px">Save</a>
									<a href="javascript:void(0)" class="easyui-linkbutton"  onclick="javascript:$('#dlgotherallow').dialog('close')" style="width:90px">Cancel</a>
									</div>
	
	
	</div>
</div>

<script>
//@SerializedName("custom_name")
$( document ).ready(function() 
{    
$('#maintab').hide();
var load_tabs=['LOAD_TAB1_CASE_MANAGEMENT.do','LOAD_TAB2_CASE_MANAGEMENT.do','LOAD_TAB3_CASE_MANAGEMENT.do',
'LOAD_TAB4_CASE_MANAGEMENT.do','LOAD_TAB5_CASE_MANAGEMENT.do','LOAD_TAB6_CASE_MANAGEMENT.do','LOAD_TAB7_CASE_MANAGEMENT.do',
'LOAD_TAB8_CASE_MANAGEMENT.do','LOAD_TAB9_CASE_MANAGEMENT.do','LOAD_TAB10_CASE_MANAGEMENT.do','LOAD_TAB11_CASE_MANAGEMENT.do'];

var save_tabs=['SAVE_TAB1_CASE_MANAGEMENT.do','SAVE_TAB2_CASE_MANAGEMENT.do','SAVE_TAB3_CASE_MANAGEMENT.do',
'SAVE_TAB4_CASE_MANAGEMENT.do','SAVE_TAB5_CASE_MANAGEMENT.do','SAVE_TAB6_CASE_MANAGEMENT.do','SAVE_TAB7_CASE_MANAGEMENT.do',
'SAVE_TAB8_CASE_MANAGEMENT.do','SAVE_TAB9_CASE_MANAGEMENT.do','SAVE_TAB10_CASE_MANAGEMENT.do','SAVE_TAB11_CASE_MANAGEMENT.do'];

var delete_tabs=['DELETE_TAB1_CASE_MANAGEMENT.do','DELETE_TAB2_CASE_MANAGEMENT.do','DELETE_TAB3_CASE_MANAGEMENT.do',
'DELETE_TAB4_CASE_MANAGEMENT.do','DELETE_TAB5_CASE_MANAGEMENT.do','DELETE_TAB6_CASE_MANAGEMENT.do','DELETE_TAB7_CASE_MANAGEMENT.do',
'DELETE_TAB8_CASE_MANAGEMENT.do','DELETE_TAB9_CASE_MANAGEMENT.do','DELETE_TAB10_CASE_MANAGEMENT.do','DELETE_TAB11_CASE_MANAGEMENT.do'];

//$('#ProgramPOCSentTo').textbox('textbox').attr('maxlength',50)
//$('#rfiNotes').textbox('textbox').attr('maxlength',500)
				
$('#ff1').form({onLoadSuccess:function(data){
		$('#ff2').form('load',data);
		if ($('#ff1loadstatus').val()=="fail")
		{
		alert($('#ff1message').val());
		}
	
	},
	onLoadError:function()
	{
		alert("Unable to retrieve the case management record");
	},
	success:function(data)
	{
	var dt=jQuery.parseJSON(data);
	if (dt)
	{
	if (dt.CaseMgmntId>0)
	{
	$('#selectedprojectid').val(dt.CaseMgmntId);
	}
	}
	}
	});
$.ajaxSetup({
  cache:false
});
});

//Load all the tabs on the tab click

$('#maintab').tabs({
onSelect:function(title,index)
{
	
	if ($('#maintab').is(":visible"))
	{
		var selected=$('#selectedprojectid').val();
		
		if (index==0)
		{
		$('#ff1').form('load',load_tabs[0]+'?projectid='+selected);
		}
		else if (index==1)
		{

		}
		else if (index==2)
		{
			$('#dg_rfi').datagrid();
			if ($('#dg_rfi').datagrid('options').url)
			{
			$('#dg_rfi').datagrid('reload'); 
			}
			else
			{
			$('#dg_rfi').datagrid({ url: load_tabs[2]+'?projectid='+selected});	
			}
		}
		else if (index==3)
		{
			$('#dgstatus').datagrid();
			if ($('#dgstatus').datagrid('options').url)
			{
				$('#dgstatus').datagrid('reload');;
			}
			else
			{
				$('#dgstatus').datagrid({ url: load_tabs[3]+'?projectid='+selected});
			}
		}
		else if (index==4)
		{
		
			$('#dgconsult').datagrid();
			if ($('#dgconsult').datagrid('options').url)
			{
				$('#dgconsult').datagrid('reload');;
			}
			else
			{
				$('#dgconsult').datagrid({ url: load_tabs[4]+'?projectid='+selected});
			}
		}
		else if (index==5)
		{
		$('#dgshpo').datagrid();
			if ($('#dgshpo').datagrid('options').url)
			{
				$('#dgshpo').datagrid('reload');;
			}
			else
			{
				$('#dgshpo').datagrid({ url: load_tabs[5]+'?projectid='+selected});
			}
		}
		else if (index==6)
		{
			$('#dgtribal').datagrid();
			if ($('#dgtribal').datagrid('options').url)
			{
				$('#dgtribal').datagrid('reload');;
			}
			else
			{
				$('#dgtribal').datagrid({ url: load_tabs[6]+'?projectid='+selected});
			}
		}
		else if (index==7)
		{
		$('#dgintletter').datagrid();
			if ($('#dgintletter').datagrid('options').url)
			{
				$('#dgintletter').datagrid('reload');;
			}
			else
			{
				$('#dgintletter').datagrid({ url: load_tabs[7]+'?projectid='+selected});
			}
		
		}
		else if (index==8)
		{
			$('#dgexorder').datagrid();
			if ($('#dgexorder').datagrid('options').url)
			{
				$('#dgexorder').datagrid('reload');;
			}
			else
			{
				$('#dgexorder').datagrid({ url: load_tabs[8]+'?projectid='+selected});
			}
		}
		else if (index==9)
		{
			$('#dgallow106').datagrid();
			if ($('#dgallow106').datagrid('options').url)
			{
				$('#dgallow106').datagrid('reload');;
			}
			else
			{
				$('#dgallow106').datagrid({ url: load_tabs[9]+'?projectid='+selected});
			}
		}
		else if (index==10)
		{
			$('#dgotherallow').datagrid();
			if ($('#dgotherallow').datagrid('options').url)
			{
				$('#dgotherallow').datagrid('reload');;
			}
			else
			{
				$('#dgotherallow').datagrid({ url: load_tabs[10]+'?projectid='+selected});
			}
		}
	}
}
});
//Load End

$("case").submit(function(event){
event.preventDefault();
});

//Save click of all the buttons
function saveform(bt)
{
var selected=$('#selectedprojectid').val();
if (bt.id=="saveform1") 
	{
	togglefields(false);
	$('#ff1').form('submit',{url:save_tabs[0]+'?CASEMGMNTID='+selected});
	togglefields(true);
	}
else if (bt.id=="saveform2") 
{
	$('#ff2').form('submit',{url:save_tabs[1]+'?CASEMGMNTID='+selected});
	var message="EHP Review/FONSI saved successfully.";
}	
else if (bt.id=="rfisave") 
{
	if ($('#rfiId').val()=='')
	{
		$('#rfiId').val(0);
	}
	$('#fmrfi').form('submit',{url:save_tabs[2]+'?CASEMGMNTID='+selected});
	var message="RFI saved successfully.";
	$('#rfidlg').dialog('close');
	$('#dg_rfi').datagrid('reload');    
	
}
else if (bt.id=="statussave") 
{
	if ($('#statusId').val()=='')
	{
		$('#statusId').val(0);
	}
	$('#fmstatus').form('submit',{url:save_tabs[3]+'?CASEMGMNTID='+selected});
	$('#dlgstatus').dialog('close');
	$('#dgstatus').datagrid('reload'); 
	
}
else if (bt.id=="consusave")
{
	if ($('#consultationId').val()=='')
	{
		$('#consultationId').val(0);
	}
	$('#fmcons').form('submit',{url:save_tabs[4]+'?CASEMGMNTID='+selected});
	$('#dlgconsult').dialog('close');
	$('#dgconsult').datagrid('reload'); 
	
	
   
}
else if (bt.id=="shposave")
{
	if ($('#shpoConsultId').val()=='')
	{
		$('#shpoConsultId').val(0);
	}
	$('#fmshpo').form('submit',{url:save_tabs[5]+'?CASEMGMNTID='+selected});
	$('#dlgshpo').dialog('close');
	$('#dgshpo').datagrid('reload'); 
}
else if (bt.id=="tribalsave")
{
	if ($('#thpoConsultId').val()=='')
	{
		$('#thpoConsultId').val(0);
	}
	$('#fmtribal').form('submit',{url:save_tabs[6]+'?CASEMGMNTID='+selected});
	$('#dlgtribal').dialog('close');
	$('#dgtribal').datagrid('reload'); 
	
}
else if (bt.id=="intlettersave")
{
	if ($('#letterId').val()=='')
	{
		$('#letterId').val(0);
	}
	$('#fmintletter').form('submit',{save_tabs[7]+'?CASEMGMNTID='+selected});
	$('#dlgintletter').dialog('close');
	$('#dgintletter').datagrid('reload'); 
}

else if (bt.id=="exordersave")
{
	
	if ($('#execOrderId').val()=='')
	{
		$('#execOrderId').val(0);
	}
	$('#fmexorder').form('submit',{url:save_tabs[8]+'?CASEMGMNTID='+selected});
	$('#dlgexorder').dialog('close');
	$('#dgexorder').datagrid('reload'); 
	
}
else if (bt.id=="allow106save")
{
	if ($('#allowanceId').val()=='')
	{
		$('#allowanceId').val(0);
	}
	$('#fmallow106').form('submit',{save_tabs[9]+'?CASEMGMNTID='+selected});
	$('#dlgallow106').dialog('close');
	$('#dgallow106').datagrid('reload'); 
	
}
else if (bt.id=="otherallowsave")
{
	if ($('#allowanceId').val()=='')
	{
		$('#allowanceId').val(0);
	}
	$('#fmotherallow').form('submit',{save_tabs[10]+'?CASEMGMNTID='+selected});
	$('#dlgotherallow').dialog('close');
	$('#dgotherallow').datagrid('reload'); 

}
/*
posting.done(function (data)
{
if (data=="Success") 
{
alert(message);
};
return false;
});*/

}

$("#projectid").change(function(){
var selectedval=$("#projectid").val();
var selected=$("#projectid").val().split('|')[0];
var ehpid=$("#projectid").val().split('|')[1];
$('#EHPId').val(ehpid);

if (selected) 
{
$('#maintab').show();
$('#ff1').form('clear');
$('#ff2').form('clear');
togglefields(true);

var dposting=$('#ff1').form('load','LOAD_TAB1_CASE_MANAGEMENT.do?projectid='+selected);
$('#selectedprojectid').val(selected);

$('#mode').val('modify');
var dmessage="Loaded.";
}
else
{
return false;
}
});

function newClick()
{
$('#ff1').form('clear');
$('#ff2').form('clear');
togglefields(false);
$('#selectedprojectid').val('0');
$('#EHPId').val('');
$('#maintab').show();
return false;
}

function togglefields(data)
{
$('#programCode').attr("DISABLED", data);
$('#disasterEventId').attr("DISABLED", data);
$('#Grantee').attr("DISABLED", data);
$('#state').attr("DISABLED", data);
$('#projectId').attr("DISABLED", data);
}

/////For the tabs
function newRFIStatus(){
$('#rfidlg').dialog('open').dialog('setTitle','Add New RFI Status');
$('#fmrfi').form('clear');
}
function editRFIStatus(){

var row = $('#dg_rfi').datagrid('getSelected');
if (row)
{
$('#rfidlg').dialog('open').dialog('setTitle','Edit RFI Status');
$('#fmrfi').form('load',row);
}
}
function removeRFIStatus(){
var selected=$('#selectedprojectid').val();
var row = $('#dg_rfi').datagrid('getSelected');
if (row)
{
$('#fmrfi').form('load',row);
$('#fmrfi').form('submit',{url:delete_tabs[3]+'?CASEMGMNTID='+selected});
$('#dg_rfi').datagrid('reload');    
}
}
function newstatus(){
$('#dlgstatus').dialog('open').dialog('setTitle','Add New Status');
$('#fmstatus').form('clear');
}
function editstatus()
{
var row = $('#dgstatus').datagrid('getSelected');
if (row)
{
$('#dlgstatus').dialog('open').dialog('setTitle','Edit Status');
$('#fmstatus').form('load',row);
}
}
function removestatus(){
var selected=$('#selectedprojectid').val();
var row = $('#dgstatus').datagrid('getSelected');
if (row)
{
$('#fmstatus').form('load',row);
$('#fmstatus').form('submit',{url:delete_tabs[4]+'?CASEMGMNTID='+selected});
$('#dgstatus').datagrid('reload');    
}
}

function newConsult(){
$('#dlgconsult').dialog('open').dialog('setTitle','Add Consultation');
$('#fmcons').form('clear');
}
function editConsult()
{
var row = $('#dgconsult').datagrid('getSelected');
if (row)
{
$('#dlgconsult').dialog('open').dialog('setTitle','Edit Consultation');
$('#fmcons').form('load',row);
}
}
function removeConsult(){
var selected=$('#selectedprojectid').val();
var row = $('#dgconsult').datagrid('getSelected');
if (row)
{
$('#fmcons').form('load',row);
$('#fmcons').form('submit',{url:delete_tabs[4]+'?CASEMGMNTID='+selected});
$('#dgconsult').datagrid('reload');    
}
}

function newShpo(){
$('#dlgshpo').dialog('open').dialog('setTitle','Add SHPO Consult');
$('#fmshpo').form('clear');
}
function editShpo()
{
var row = $('#dgshpo').datagrid('getSelected');
if (row)
{
$('#dlgshpo').dialog('open').dialog('setTitle','Edit SHPO Consult');
$('#fmshpo').form('load',row);
}
}
function removeShpo(){
var selected=$('#selectedprojectid').val();
var row = $('#dgshpo').datagrid('getSelected');
if (row)
{
$('#fmshpo').form('load',row);
$('#fmshpo').form('submit',{url:delete_tabs[5]+'?CASEMGMNTID='+selected});
$('#dgshpo').datagrid('reload');    
}
}
function newTribal(){
$('#dlgtribal').dialog('open').dialog('setTitle','Add Tribal Consult');
$('#fmtribal').form('clear');
}
function editTribal()
{
var row = $('#dgtribal').datagrid('getSelected');
if (row)
{
$('#dlgtribal').dialog('open').dialog('setTitle','Edit Tribal Consult');
$('#fmtribal').form('load',row);
}
}
function removeTribal(){
var selected=$('#selectedprojectid').val();
var row = $('#dgtribal').datagrid('getSelected');
if (row)
{
$('#fmtribal').form('load',row);
$('#fmtribal').form('submit',{url:delete_tabs[6]+'?CASEMGMNTID='+selected});
$('#dgtribal').datagrid('reload');    
}
}



function newLetter(){
$('#dlgintletter').dialog('open').dialog('setTitle','Add Internal Letter');
$('#fmintletter').form('clear');
}
function editLetter()
{
var row = $('#dgintletter').datagrid('getSelected');
if (row)
{
$('#dlgintletter').dialog('open').dialog('setTitle','Edit Internal Letter');
$('#fmintletter').form('load',row);
}
}
function removeLetter(){
var selected=$('#selectedprojectid').val();
var row = $('#dgintletter').datagrid('getSelected');
if (row)
{
$('#fmintletter').form('load',row);
$('#fmintletter').form('submit',{url:delete_tabs[7]+'?CASEMGMNTID='+selected});
$('#dgintletter').datagrid('reload');    
}
}
function newExorder(){
$('#dlgexorder').dialog('open').dialog('setTitle','Add Executive Order');
$('#fmexorder').form('clear');
}
function editExorder()
{
var row = $('#dgexorder').datagrid('getSelected');
if (row)
{
$('#dlgexorder').dialog('open').dialog('setTitle','Edit Executive Order');
$('#fmexorder').form('load',row);
}
}
function removeExorder(){
var selected=$('#selectedprojectid').val();
var row = $('#dgexorder').datagrid('getSelected');
if (row)
{
$('#fmexorder').form('load',row);
$('#fmexorder').form('submit',{url:delete_tabs[8]+'?CASEMGMNTID='+selected});
$('#dgexorder').datagrid('reload');    
}
}
function newallow106(){
$('#dlgallow106').dialog('open').dialog('setTitle','Add Allowance(Section 106)');
$('#fmallow106').form('clear');
}
function editallow106()
{
var row = $('#dgallow106').datagrid('getSelected');
if (row)
{
$('#dlgallow106').dialog('open').dialog('setTitle','Edit Allowance(Section 106)');
$('#fmallow106').form('load',row);
}
}
function removeallow106(){
var selected=$('#selectedprojectid').val();
var row = $('#dgallow106').datagrid('getSelected');
if (row)
{
$('#fmallow106').form('load',row);
$('#fmallow106').form('submit',{url:delete_tabs[9]+'?CASEMGMNTID='+selected});
$('#dgallow106').datagrid('reload');    
}
}
function newotherallow(){
$('#dlgotherallow').dialog('open').dialog('setTitle','Add Other Allowance');
$('#fmotherallow').form('clear');
}
function editotherallow()
{
var row = $('#dgotherallow').datagrid('getSelected');
if (row)
{
$('#dlgotherallow').dialog('open').dialog('setTitle','Edit Other Allowance');
$('#fmotherallow').form('load',row);
}
}
function removeotherallow(){
var selected=$('#selectedprojectid').val();
var row = $('#dgotherallow').datagrid('getSelected');
if (row)
{
$('#fmotherallow').form('load',row);
$('#fmotherallow').form('submit',{url:delete_tabs[10]+'?CASEMGMNTID='+selected});
$('#dgotherallow').datagrid('reload');    
}
}
/////
/*
        var a=document.getElementsByName("dd")[0].value;
            document.getElementsByName("sow")[0].previousSibling.setAttribute("maxlength",10)
            $( "select" )
  .change(function () {
    var str = "";
    $( "select option:selected" ).each(function() {
      str += $( this ).text() + " ";
    });
    $( "div" ).text( str );
    https://blog.oio.de/2010/11/08/how-to-create-a-loading-animation-spinner-using-jquery/
    http://jsfiddle.net/qb7cR/
*/
</script>

