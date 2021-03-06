DATAPATH = $(top_srcdir)/data

pkgdata_DATA = GL27 nad.lst proj_def.dat nad27 nad83 world other.extra \
		CH \
		ITRF2000 ITRF2008 ITRF2014 proj.db

SQL_ORDERED_LIST = sql/begin.sql \
		sql/proj_db_table_defs.sql \
		sql/conversion_triggers.sql \
		sql/metadata.sql \
		sql/unit_of_measure.sql \
		sql/area.sql \
		sql/coordinate_system.sql \
		sql/axis.sql \
		sql/ellipsoid.sql \
		sql/prime_meridian.sql \
		sql/geodetic_datum.sql \
		sql/vertical_datum.sql \
		sql/conversion.sql \
		sql/geodetic_crs.sql \
		sql/projected_crs.sql \
		sql/vertical_crs.sql \
		sql/compound_crs.sql \
		sql/helmert_transformation.sql \
		sql/grid_transformation.sql \
		sql/grid_transformation_custom.sql \
		sql/other_transformation.sql \
		sql/concatenated_operation.sql \
		sql/alias_name.sql \
		sql/supersession.sql \
		sql/deprecation.sql \
		sql/esri.sql \
		sql/ignf.sql \
		sql/grid_alternatives.sql \
		sql/grid_alternatives_generated.sql \
		sql/customizations.sql \
		sql/commit.sql

EXTRA_DIST = GL27 nad.lst proj_def.dat nad27 nad83 \
		world other.extra \
		CH \
		ITRF2000 ITRF2008 ITRF2014 \
		CMakeLists.txt tests/test_nodata.gtx null.lla \
		generate_all_sql_in.cmake sql_filelist.cmake \
		$(SQL_ORDERED_LIST)

process-nad2bin:
	@if [ -f $(DATAPATH)/null.lla -a ! -f null ] || [ -f $(DATAPATH)/conus.lla -a ! -f conus ] ; then \
	  for x in $(DATAPATH)/*.lla ; do \
	    DEST=`basename $$x .lla`; \
	    echo "../src/nad2bin$(EXEEXT) $$DEST < $$x"; \
	    ../src/nad2bin$(EXEEXT) $$DEST < $$x ; \
	  done; \
	fi

install-data-local: process-nad2bin
	$(mkinstalldirs) $(DESTDIR)$(pkgdatadir)
	@if [ -f $(DATAPATH)/null.lla -a ! -f $(DATAPATH)/null ] || [ -f $(DATAPATH)/conus.lla -a ! -f $(DATAPATH)/conus ] ; then \
	  for x in $(DATAPATH)/*.lla ; do \
	    DEST=$(DESTDIR)$(pkgdatadir)/`basename $$x .lla`; \
	    echo "../src/nad2bin$(EXEEXT) $$DEST < $$x"; \
	    ../src/nad2bin$(EXEEXT) $$DEST < $$x ; \
	    if [ $$? -ne 0 ] ; then \
		echo "nad2bin$(EXEEXT) failed to install $$DEST"; \
		exit 1; \
	    fi; \
	  done; \
	else \
	  echo "nad2nad NADCON source files not present"; \
	fi
	@for gridfile in $(DATAPATH)/*.gsb $(DATAPATH)/*.gtx $(DATAPATH)/ntv1_can.dat dummy \
	                 $(DATAPATH)/alaska $(DATAPATH)/conus $(DATAPATH)/hawaii $(DATAPATH)/null \
	                 $(DATAPATH)/prvi $(DATAPATH)/stgeorge $(DATAPATH)/stlrnc $(DATAPATH)/stpaul \
	                 $(DATAPATH)/FL $(DATAPATH)/MD $(DATAPATH)/TN $(DATAPATH)/WI $(DATAPATH)/WO; do \
	  if test "$$gridfile" != "dummy" -a -f "$$gridfile" ; then \
	    echo $(INSTALL_DATA) $$gridfile $(DESTDIR)$(pkgdatadir)/`basename $$gridfile`; \
	    $(INSTALL_DATA) $$gridfile $(DESTDIR)$(pkgdatadir)/`basename $$gridfile`; \
          fi; \
	done

proj.db: $(DATAPATH)/sql/*.sql
	@echo "Make proj.db"
	$(RM) proj.db
	@export SQL_EXPANDED_LIST=""; \
	 for x in $(SQL_ORDERED_LIST); do \
		export SQL_EXPANDED_LIST="$${SQL_EXPANDED_LIST} $(DATAPATH)/$$x"; \
	 done; \
	 cat $${SQL_EXPANDED_LIST} | sqlite3 proj.db; \
	 if [ $$? -ne 0 ] ; then \
		echo "Build of proj.db failed"; \
		$(RM) proj.db; \
		exit 1; \
	fi; \
	 if [[ $$(echo "pragma foreign_key_check;" | sqlite3 proj.db | head -c1 | wc -c) -ne 0 ]]; then \
		echo "Foreign key check failed"; \
		$(RM) proj.db; \
		exit 1; \
	 fi

	
# For out-of-tree builds, link all file of the source data dir to the generated data
check-local: process-nad2bin
	@if [ ! -f GL27 ]; then \
		for x in $(DATAPATH)/*; do \
			ln -sf $$x .; \
		done \
	fi
