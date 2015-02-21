---
layout: post
title: Function references for common code parts
---

```java


public interface IdGeneration3rdPtyService {
   String generateBookId();

   String generateAuthorId();

   /* ... more generator */
}

public class IdGenerationExecutor implements Loggable {

    @Autowired
    IdGeneration3rdPtyService idGenerationService;

    public String generateAuditId(){
        return generateIdAndHandleAnyException(idGeneratorGateway::getAuditId);
    }

    public String generateAuditFindingId(){
        return generateIdAndHandleAnyException(idGeneratorGateway::getAuditFindingId);
    }

    public String generateAuditReportId(){
        return generateIdAndHandleAnyException(idGeneratorGateway::getAuditReportId);
    }

    public String generateCorrectiveActionId(){
        return generateIdAndHandleAnyException(idGeneratorGateway::getCorrectiveActionId);
    }

    private String generateIdAndHandleAnyException(Supplier<String> idSupplierFunction){
        IdDTO generatedId = null;
        try {
            logger().debug("Generate an id");
            generatedId = idSupplierFunction.get();
        } catch(GatewayException ex){
            throw new RestException(RestErrors.ID_GENERATION_ERROR.toRestError(), RestUtils.createParams(ParamNameValue.toParamNameValue("Error", "No response from gateway")));
        }
        if(generatedId == null){
            throw new RestException(RestErrors.ID_GENERATION_ERROR.toRestError(), RestUtils.createParams(ParamNameValue.toParamNameValue("Error", "Returned id is null")));
        }
        if(Strings.isNullOrEmpty(generatedId.getCode())){
            throw new RestException(RestErrors.ID_GENERATION_ERROR.toRestError(), RestUtils.createParams(ParamNameValue.toParamNameValue("Error", "Returned code is null")));
        }
        return generatedId.getCode();
    }
}
```


T.