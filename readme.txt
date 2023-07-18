package con.xx.sync.processor;

import com.siebel.data.SiebelPropertySet;
import com.xx.chassis.logging.slf4j.Logger;
import com.xx.chassis.logging.slf4j.LoggerFactory;
import com.xx.sync.dto.SyncRequest;
import com.xx.sync.model.*;
import com.xx.sync.siebel.queries.RAMSiebelServiceMethod;
import con.xx.sync.digester.SiebelAdminProfileDigester;
import con.xx.sync.exceptions.InvalidMessageException;
import con.xx.sync.service.*;
import ong.springFramework.core.ParameterizedTypeReference;
import ong.springfrasework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

import static com.xx.sync.utils.DataUtility.replaceIfNull;
import static com.xx.sync.utils.IDMConstants.*;

@Service("addOrUpdateAdmin")
public class UpdateAdminProcessor implements MessageProcessor {

    private static final String STATUS_CODE_ADD = "C";
    private static final String STATUS_CODE_REMOVE = "D";

    private final Logger log = LoggerFactory.getLogger(UpdateAdminProcessor.class);

    @Autowired
    private UserService userService;

    private String userName;

    @Override
    public boolean validate(SyncRequest request) throws InvalidMessageException {
        log.debug("Entering validate() - {}", request);
        userName = Optional.ofNullable(request.getMessagePayLoad().get(USER_NAME))
                .map(Object::toString)
                .orElseThrow(() -> new InvalidMessageException("Invalid request; userName not found"));

        log.info("Validate request completed");
        return true;
    }

    @Override
    public String process(SyncRequest request, IdmService idmService, RAMSiebelService siebelService) throws Exception {
        IdmAdminProfile idmResponse = idmService.get(
                CON_CONFIG_ADMIN_PROFILE_ENDPOINT + userName,
                new ParameterizedTypeReference<IdmAdminProfile>() {},
                true);

        log.info("IDM Response: {}", idmResponse);

        IdmAdminProfileResult adminProfileResultResponse = idmResponse.getResult();

        String adminType = replaceIfNull(adminProfileResultResponse.getAuthRole(), "");

        List<IdmOrganization> addAdminOfOrgs = filterOrganizationsByStatusCode(adminProfileResultResponse.getAdminOfOrgs(), STATUS_CODE_ADD);
        List<IdmOrganization> removeAdminOfOrgs = filterOrganizationsByStatusCode(adminProfileResultResponse.getAdminOfOrgs(), STATUS_CODE_REMOVE);
        List<IdmOrganization> addOwnerOfOrgs = filterOrganizationsByStatusCode(adminProfileResultResponse.getUaManagedApps(), STATUS_CODE_ADD);
        List<IdmOrganization> removeOwnerOfOrgs = filterOrganizationsByStatusCode(adminProfileResultResponse.getUaManagedApps(), STATUS_CODE_REMOVE);

        AdminProfileUpdateRequest adminProfileUpdateRequest = new AdminProfileUpdateRequest();
        adminProfileUpdateRequest.setAddOrgs(addAdminOfOrgs);
        adminProfileUpdateRequest.setRemoveOrgs(removeAdminOfOrgs);
        adminProfileUpdateRequest.setAddOrganizations(addOwnerOfOrgs);

        String roleName = adminType.equals("CA") ? "CorpAdmin" : "UserAdmin";

        RAMSiebelServiceMethod ramSiebelServiceMethod = adminType.equals("CA")
                ? RAMSiebelServiceMethod.UPSERT_CORP_ADMIN_PROFILE
                : RAMSiebelServiceMethod.UPSERT_USER_ADMIN_PROFILE;

        User user = userService.getUser(userName);

        SiebelPropertySet siebelPropertySet = SiebelAdminProfileDigester.getPropertySetByType(user, adminProfileUpdateRequest, roleName);
        SiebelPropertySet outputPS = siebelService.executeUpsert(ramSiebelServiceMethod, siebelPropertySet);

        boolean isSiebelCallSuccessful = SiebelAdminProfileDigester.verifyResultSiebelPropertySet(outputPS);

        log.info("For userId {}, the update is {}", userName, isSiebelCallSuccessful ? "Successful" : "Unsuccessful");
        log.debug("Result of the admin profile update: {}", outputPS.toString());

        log.info("Completed the Admin Profile update");
        return null; // TODO: Add a meaningful return value
    }

    @Override
    public void postProcess() {
        // TODO: Add post-processing logic if needed
    }
    
    private List<IdmOrganization> filterOrganizationsByStatusCode(List<? extends IdmOrganization> organizations, String statusCode) {
        List<IdmOrganization> filteredOrganizations = new ArrayList<>();
        for (IdmOrganization organization : organizations) {
            if (organization != null && statusCode.equals(replaceIfNull(organization.getStatusCode(), ""))) {
                filteredOrganizations.add(organization);
            }
        }
        return filteredOrganizations;
    }
}

